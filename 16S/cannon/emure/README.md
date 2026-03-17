# EMU Reference-Based OTU97 Pipeline (EMURE)

**Reference-based 97% OTU clustering for ONT full-length 16S rRNA amplicons using EMU + VSEARCH**

This pipeline takes the EMU expectation-maximisation taxonomic classification as its starting point, then clusters the classified taxa into 97% identity OTUs by comparing their representative reference sequences from the MIMt database. It is the complement of the [de novo pipeline](../denovo/README.md) which clusters raw reads first and assigns taxonomy afterward.

**Last updated:** 2026-03-06
**Location:** `/home/uca/cousin/sixteen/cannon/emure/`
**Conda environment:** `sixteen` (vsearch 2.30.1, seqkit, python 3)

---

## Quick Start

```bash
conda activate sixteen
cd /home/uca/cousin/sixteen/cannon/emure

# Full pipeline (EMU table → collapser → VSEARCH cluster → OTU tables)
bash scripts/run_emure.sh

# Or just regenerate wide/long tables from existing collapser output
python3 scripts/build_otu_tables.py

# Threepoint reproducibility filter (run from threepoint/ directory)
cd /home/uca/cousin/sixteen/scripts/threepoint
python3 filter_threepoint_emure.py
```

---

## Methodology

### Overview: Reference-Based vs De Novo Clustering

The two 16S analysis pipelines in this project take fundamentally different approaches to OTU construction:

| Step | **EMURE (this pipeline)** | **De Novo** (`denovo/`) |
|------|---------------------------|------------------------|
| **Step 1** | EMU classifies each read against MIMt (EM algorithm) | Raw reads QC'd with porechop + chopper |
| **Step 2** | Extract representative reference sequences for detected taxa | Pool reads, orient against MIMt |
| **Step 3** | Cluster reference sequences at 97% identity (VSEARCH) | 99% pseudo-denoising (collapse error variants) |
| **Step 4** | Aggregate EMU abundances by cluster | Chimera removal (uchime_ref vs MIMt) |
| **Step 5** | Generate OTU tables with parsed taxonomy | 97% OTU clustering of denoised centroids |
| **Step 6** | — | Dual taxonomy (SINTAX + usearch_global) |
| **Input to clustering** | 436 MIMt reference sequences (for detected taxa) | 285,354 QC'd amplicon reads |
| **OTU count** | 353 | 857 |
| **Singletons** | 299 (84.7%) | 0 (0%) |
| **Taxonomy source** | EMU EM classification (pre-assigned) | SINTAX k-mer + BLAST best-hit (post-assigned) |
| **Novel taxa** | Invisible (must be in MIMt) | Detectable (reads cluster regardless of reference) |

### Why Both Pipelines?

**EMURE strengths:**
- EMU's EM algorithm provides statistically principled abundance estimates (fractional read assignment to closely related taxa)
- Taxonomy is assigned at read level before clustering, ensuring every OTU has a confident classification
- Fast: only 436 reference sequences need clustering, not 285k reads
- Integrated with the established EMU-based workflow used in the main thesis analysis

**EMURE limitations:**
- Reference-dependent: taxa not in MIMt are invisible to EMU and therefore absent from this pipeline
- High singleton rate (84.7%) reflects that most detected species have reference sequences >3% divergent from their nearest neighbour — they cannot merge
- No error correction or chimera removal: relies entirely on EMU's read-level filtering
- Clusters reference sequences, not actual sample sequences — centroid sequences are database entries, not sample-derived

**De novo strengths:**
- Reference-independent: novel taxa cluster into OTUs even without database matches
- Two-step denoising eliminates ONT error variants before clustering
- Zero singletons (minimum 3 supporting reads per 99% cluster, minimum 9 reads per OTU)
- Centroid sequences are real sample reads (suitable for phylogenetic analysis, BLAST confirmation)

Both pipelines are applied to the same 6 samples (SS1: BC13-15; SS2: BC16-18) from the TCDD-contaminated anonymized contaminated site and use the same MIMt 16S reference database (31,426 sequences).

---

## Pipeline Steps

### Step 1: EMU Abundance Table (Input)

**Source:** `sixteen/cannon/emu/emu_abundance.tsv`

EMU (Curry et al. 2022) is a long-read-aware taxonomic classification framework designed for Oxford Nanopore 16S sequences. It aligns each read against the MIMt reference database using a modified Minimap2, then applies an expectation-maximisation (EM) algorithm to resolve ambiguous multi-mapping reads. The EM step iteratively re-weights taxon abundances until convergence, producing fractional read assignments (`estimated_counts`) that represent the statistically most likely distribution of reads across taxa.

**Key output columns:**
- `sample_id` — barcode identifier (e.g., `barcode13_120_filtered`)
- `tax_id` — NCBI taxonomy ID for the assigned species
- `abundance` — relative abundance (sums to 1.0 per sample)
- `estimated_counts` — fractional read count (total ≈ reads per sample)
- Full taxonomic lineage: `superkingdom` through `species`

**Result:** 436 unique species detected across 6 samples (250 genera)

### Step 2: Collapser Format Conversion

**Script:** `scripts/build_16s_collapser.py`
**Output:** `01_collapser/collapser_input.tsv`

Converts the EMU table to a standardised "collapser" format with explicit taxonomy columns (Kingdom, Phylum, Class, Order, Family, Genus, Species) and the NCBI tax ID. No filtering is applied — all 436 taxa from EMU are preserved regardless of abundance or prevalence. The `estimated_counts` field becomes `read_count`.

### Step 3: Representative Sequence Extraction

**Output:** `02_reference_seqs/`

For each of the 436 detected tax IDs, a representative 16S sequence is extracted from the MIMt reference database using the `mimt_mapping.fixed.tsv` lookup table (maps NCBI tax ID → MIMt sequence accession). These sequences are the actual reference entries from MIMt — not sample reads.

**Files produced:**
- `taxid2seq.tsv` — Tax ID → MIMt sequence accession mapping (436 rows)
- `rep_seq_ids.txt` — List of 436 accession IDs
- `rep_seqs.fasta` — Extracted FASTA sequences (436 entries, seqkit grep)

### Step 4: VSEARCH Reference-Sequence Clustering at 97% Identity

**Script:** `scripts/collapser_otu97.sh`
**Output:** `03_cluster/`

```bash
vsearch --cluster_fast rep_seqs.fasta --id 0.97 --centroids centroids_0.97.fasta --uc clusters_0.97.uc
```

The 436 representative MIMt sequences are clustered at 97% identity. This groups species whose 16S reference sequences are ≥97% similar — the traditional species-level OTU threshold.

**Clustering results:**

| Metric | Value |
|--------|-------|
| Input sequences | 436 |
| Output clusters (OTUs) | 353 |
| Merged sequences | 83 (sequences absorbed into clusters) |
| Singleton clusters | 299 (84.7% — single-species OTUs) |
| Multi-species clusters | 54 (15.3% — 2+ species per OTU) |

The high singleton rate is expected and correct: most bacterial species have >3% divergence in their 16S rRNA gene from their nearest neighbour, so their reference sequences cannot cluster with any other detected species. The 54 multi-species clusters represent genuinely closely related species pairs (e.g., *Clostridium subterminale* + *Clostridium culturomicium*, or *Sphingomonas gilva* + *Sphingomonas cavernae*).

**Files produced:**
- `clusters_0.97.uc` — VSEARCH universal cluster format (353 S-records + 83 H-records)
- `centroids_0.97.fasta` — Centroid reference sequences (353 entries)
- `seq2cluster.tsv` — Sequence accession → cluster ID mapping
- `taxid2cluster.tsv` — NCBI tax ID → cluster ID mapping (436 rows)

### Step 5: Abundance Aggregation by Cluster

The final step in `collapser_otu97.sh` aggregates the per-taxon, per-sample abundances from Step 2 by cluster membership. For each sample × cluster combination:
- `abundance` = sum of EMU relative abundances for all member taxa
- `read_count` = sum of EMU estimated_counts for all member taxa
- `member_taxids` = semicolon-separated NCBI tax IDs
- `member_taxa` = semicolon-separated lineage strings (`taxid||Phylum|Class|Order|Family|Genus|Species`)

**Output:** `04_tables/collapser_abundance_0.97.tsv` (1,381 rows: 353 clusters × ~6 samples, minus absent clusters)

### Step 6: Wide/Long OTU Table Generation

**Script:** `scripts/build_otu_tables.py`
**Output:** `04_tables/otu_table_combined.tsv`, `04_tables/otu_table_long.tsv`

Transforms the long-format collapser table into standardised wide and long formats matching the de novo pipeline output:

**Wide table** (`otu_table_combined.tsv`):
```
OTU_ID  total_reads  barcode13  barcode14  barcode15  barcode16  barcode17  barcode18  member_count  Kingdom  Phylum  Class  Order  Family  Genus  Species
OTU_119 26916.1      6635.3     4164.8     5194.5     2643.7     2463.2     5814.6     1            Bacteria Bacillota Clostridia ...  Fonticella  Fonticella tunisiensis
```

**Taxonomy assignment for multi-member clusters:** For the 54 clusters containing 2+ species, taxonomy is assigned from the **dominant member** — the taxon with the highest total estimated_counts across all samples, as determined from the `composition_checker_0.97.tsv` quality assurance table. This is appropriate because EMU has already provided confident species-level assignments; the dominant member represents the primary biological signal.

**Long table** (`otu_table_long.tsv`): One row per OTU × barcode, with columns for `relative_abundance` (within-sample) and `cohort` (SS1/SS2).

---

## Threepoint Reproducibility Filter

**Script:** `threepoint/filter_threepoint_emure.py`
**Output:** `threepoint/outputs/emure/`

The threepoint filter enforces biological reproducibility by requiring that an OTU has **≥10 estimated reads in ALL 3 biological replicates** of at least one soil sample. This is identical to the threshold applied to the de novo pipeline.

**Sample structure:**
- SS1 (Soil Sample 1): barcodes 13, 14, 15 — three biological replicates
- SS2 (Soil Sample 2): barcodes 16, 17, 18 — three biological replicates

**Filter logic:** An OTU passes if `min(barcode_reads) ≥ 10.0` across all three replicates of SS1, SS2, or both.

**Note on EMU estimated_counts:** Unlike the de novo pipeline where read counts are integers (mapped read totals), EMU's `estimated_counts` are fractional values reflecting the EM algorithm's probabilistic read assignment. A count of 17.1 means EMU assigned approximately 17 reads to that taxon in that sample, allowing fractional assignment of ambiguous reads. The ≥10 threshold is applied to these fractional values using float comparison.

### Emure Threepoint Results

| Metric | Emure | De Novo (for comparison) |
|--------|-------|--------------------------|
| Input OTUs | 353 | 857 |
| Passing OTUs | 245 (69.4%) | 357 (41.7%) |
| Removed OTUs | 108 (30.6%) | 500 (58.3%) |
| Reads retained | 276,635 / 279,969 (98.8%) | 227,834 / 242,557 (93.9%) |
| Both cohorts | 125 | 120 |
| SS1 only | 30 | 101 |
| SS2 only | 90 | 136 |

The higher pass rate for emure (69.4% vs 41.7%) reflects EMU's pre-filtering: EMU only reports taxa it confidently detects, so most survive the reproducibility threshold. The de novo pipeline retains more total OTUs (857 vs 353) because it captures novel taxa and error-corrected variants that EMU discards, but a larger fraction of these are low-abundance or sample-specific.

---

## Key Metrics Summary

| Metric | Value |
|--------|-------|
| EMU input species | 436 |
| EMU input genera | 250 |
| Representative MIMt sequences | 436 |
| OTU clusters (97% identity) | 353 |
| Singleton clusters (1 species) | 299 (84.7%) |
| Multi-species clusters | 54 (15.3%) |
| Total estimated reads | 279,969 |
| Threepoint-passing OTUs | 245 (69.4%) |
| Reads retained after threepoint | 276,635 (98.8%) |

### Top 10 OTUs by Total Estimated Reads

| OTU | Total Reads | Genus | Species |
|-----|-------------|-------|---------|
| OTU_119 | 26,916 | *Fonticella* | *F. tunisiensis* |
| OTU_247 | 18,635 | *Xylanivirga* | *X. thermophila* |
| OTU_125 | 9,646 | *Oxobacter* | *O. pfennigii* |
| OTU_10 | 9,631 | *Thermoclostridium* | *T. caenicola* |
| OTU_316 | 9,249 | *Acidilutibacter* | *A. cellobiosedens* |
| OTU_246 | 8,723 | *Aliitabrizicola* | *A. rongguiensis* |
| OTU_294 | 8,536 | *Lutispora* | *L. thermophila* |
| OTU_275 | 8,443 | *Acetivibrio* | *A. straminisolvens* |
| OTU_189 | 7,498 | *Hephaestia* | *H. mangrovi* |
| OTU_170 | 5,955 | *Macellibacteroides* | *M. fermentans* |

### Phylum Distribution (353 OTUs)

| Phylum | OTUs |
|--------|------|
| Bacillota | 156 (44.2%) |
| Pseudomonadota | 132 (37.4%) |
| Bacteroidota | 25 (7.1%) |
| Actinomycetota | 5 (1.4%) |
| Verrucomicrobiota | 5 (1.4%) |
| Other (10 phyla) | 30 (8.5%) |

---

## Directory Structure

```
emure/
├── 00_emu_input/                    # Step 1: EMU abundance table
│   └── emu_abundance.tsv            # Symlink → sixteen/cannon/emu/emu_abundance.tsv
├── 01_collapser/                    # Step 2: Format conversion
│   └── collapser_input.tsv          # Standardised collapser format (436 taxa × 6 samples)
├── 02_reference_seqs/               # Step 3: MIMt representative sequences
│   ├── taxid2seq.tsv                # Tax ID → accession mapping
│   ├── rep_seq_ids.txt              # 436 accession IDs
│   └── rep_seqs.fasta               # Extracted reference sequences
├── 03_cluster/                      # Step 4: VSEARCH 97% clustering
│   ├── clusters_0.97.uc             # Universal cluster format
│   ├── centroids_0.97.fasta         # 353 centroid sequences
│   ├── seq2cluster.tsv              # Accession → cluster ID
│   └── taxid2cluster.tsv            # Tax ID → cluster ID (436 rows)
├── 04_tables/                       # Steps 5-6: Final abundance tables
│   ├── otu_table_combined.tsv       # Wide format (353 OTUs × 6 barcodes + taxonomy)
│   ├── otu_table_long.tsv           # Long format (2,118 rows)
│   ├── collapser_abundance_0.97.tsv # Original collapser format (1,381 rows)
│   └── composition_checker_0.97.tsv # QA lineage table (per-member taxonomy)
├── scripts/                         # Pipeline scripts
│   ├── run_emure.sh                 # Main orchestrator
│   ├── collapser_otu97.sh           # VSEARCH clustering pipeline
│   ├── build_16s_collapser.py       # EMU → collapser format converter
│   ├── collapser_cluster_memberships.py  # QA table generator
│   └── build_otu_tables.py          # Wide/long OTU table generator
├── otu97.log                        # Execution log
└── README.md                        # This documentation
```

---

## Output File Descriptions

### Primary Analysis Tables (`04_tables/`)

| File | Format | Rows | Description |
|------|--------|------|-------------|
| `otu_table_combined.tsv` | Wide TSV | 353 | One row per OTU. Columns: OTU_ID, total_reads, barcode13-18, member_count, Kingdom-Species. Sorted by total_reads descending. |
| `otu_table_long.tsv` | Long TSV | 2,118 | One row per OTU × barcode. Additional columns: relative_abundance, cohort. |
| `collapser_abundance_0.97.tsv` | Long TSV | 1,381 | Original collapser format with member_taxids and member_taxa fields. Preserved for backward compatibility with `compare_emure_denovo.py`. |
| `composition_checker_0.97.tsv` | Long TSV | 1,643 | Per-member taxonomy table. Each row is one taxon within one cluster in one sample. Used for QA and for resolving multi-member cluster taxonomy. |

### Threepoint-Filtered Tables (`threepoint/outputs/emure/`)

| File | Format | Rows | Description |
|------|--------|------|-------------|
| `emure_otu97_threepoint.tsv` | Wide TSV | 245 | Passing OTUs only (≥10 reads in all 3 reps of ≥1 soil sample) |
| `emure_otu97_threepoint_long.tsv` | Long TSV | 1,470 | Long format of passing OTUs (245 × 6 barcodes) |
| `emure_otu97_threepoint_summary.tsv` | Wide TSV | 353 | All OTUs with pass/fail status, min per-cohort reads |

---

## Race Visualization Integration

The emure tables are loadable from the race/claude R visualization framework via functions in `race/claude/scripts/utils_claude.R`:

```R
source("scripts/utils_claude.R")

# Raw emure OTU tables
emure_wide <- load_emure_wide()           # 353 OTUs, wide format
emure_long <- load_emure_long()           # 2,118 rows, long format

# Threepoint-filtered tables
emure_tp_wide <- load_emure_threepoint_wide()   # 245 OTUs
emure_tp_long <- load_emure_threepoint_long()   # 1,470 rows
```

These follow the same conventions as existing loaders (`load_denovo_wide()`, `load_denovo_long()`) with automatic cohort assignment and barcode labelling.

---

## Scientific Context

This pipeline implements the reference-based OTU clustering approach described in the thesis:

> *"Taxonomic assignment of nanopore metabarcoding reads was performed using the EMU pipeline, a long-read-aware framework specifically designed for microbial classification from Oxford Nanopore sequencing data (Curry et al. 2022). EMU integrates read filtering, long-read alignment, and abundance estimation within a single workflow... EMU resolves between similar scoring alignments using an expectation-maximisation (EM) algorithm to estimate taxon abundances."*

The 97% OTU clustering threshold is the standard prokaryotic species delineation level (Stackebrandt & Goebel, 1994; updated by Yarza et al., 2014). At this threshold:

- Sequences sharing ≥97% 16S rRNA identity are considered conspecific or closely related
- The 83 merging events (436 → 353 clusters) represent genuinely close species pairs
- The 299 singleton clusters represent species with no near-neighbour among the other detected taxa

### Database: MIMt 16S

The Microbial Identification using MinION Technology (MIMt) database contains 31,426 full-length 16S rRNA sequences with NCBI taxonomy annotations. It was specifically curated for long-read nanopore classification and provides the reference backbone for both EMU classification and representative sequence extraction.

### Comparison with Established Collapser (95% OTU)

The original thesis analysis also clustered EMU taxa at 95% identity (genus-level OTUs), producing 258 clusters (192 passing threepoint). The 95% approach was preferred in the main analysis because:
1. Mixed-genus clusters were rare at 95%, making cluster identity straightforward
2. EMU already resolved genus identity, so the 95% grouping added little ambiguity
3. The 97% clusters introduced species-level splitting that was informative but harder to interpret

This 97% emure pipeline provides the complementary species-level view, now with proper threepoint filtering and standardised table formats for direct comparison with the de novo pipeline.

---

## Dependencies

- **vsearch** 2.30.1+ (clustering)
- **seqkit** (sequence extraction)
- **python** 3.10+ (table generation)
- **conda env:** `sixteen`

## References

- Curry KD et al. (2022) Emu: Species-Level Microbial Community Profiling of Full-Length 16S rRNA Oxford Nanopore Sequencing Data. *Nature Methods* 19, 845–853.
- Rognes T et al. (2016) VSEARCH: A Versatile Open Source Tool for Metagenomics. *PeerJ* 4, e2584.
- Stackebrandt E & Goebel BM (1994) Taxonomic Note: A Place for DNA-DNA Reassociation and 16S rRNA Sequence Analysis in the Present Species Definition in Bacteriology. *Int J Syst Bacteriol* 44, 846–849.
- Yarza P et al. (2014) Uniting the Classification of Cultured and Uncultured Bacteria and Archaea Using 16S rRNA Gene Sequences. *Nature Reviews Microbiology* 12, 635–645.
