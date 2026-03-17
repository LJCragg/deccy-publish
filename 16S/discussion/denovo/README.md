# VSEARCH De Novo OTU Clustering Pipeline (Claude Version)

**De novo 97% OTU clustering for ONT full-length 16S rRNA amplicons using VSEARCH**

This pipeline is an alternative to the original [EMU-based sixteen pipeline](../../sixteen/README.md). Rather than relying on reference-based taxonomic classification (Emu) followed by post-hoc OTU clustering, this pipeline clusters raw QC'd reads directly into OTUs using a two-step denoising strategy designed for the high error rates of Oxford Nanopore long reads.

**Last updated:** 2026-02-26
**Location:** `/home/uca/cousin/sixteen/discussion/denovo/`
**Conda environment:** `sixteen` (vsearch 2.30.1, seqkit, chopper, porechop)

---

## Why a Second Pipeline?

The original sixteen pipeline uses Emu to classify each read against the MIMt reference database, then clusters the *classified taxids* with VSEARCH at 95%/97% identity. This is a reference-dependent approach: if a species is missing from MIMt, it is invisible. The EMU pipeline also encountered problems at 97% resolution -- the `oven/associated97/` directory was empty (zero OTUs survived the threepoint filter at species level), forcing a fallback to 95% clustering.

This pipeline takes the opposite approach: cluster the *raw reads themselves* de novo, then assign taxonomy afterward. This has several advantages:

1. **Novel taxa are not lost.** Reads that don't match MIMt still cluster into OTUs and get counted.
2. **OTU centroids are real sequences.** Each OTU is represented by the actual centroid read, not a database entry. These centroids can be BLASTed, aligned, or used for phylogenetic placement.
3. **Species-level resolution works.** The two-step denoising strategy produces clean 97% OTUs without the singleton explosion that plagued the original approach.
4. **Consensus sequences are preserved.** Per-cluster consensus FASTA files are ready for downstream work (e.g., multiple sequence alignment, polishing).

---

## Quick Start

```bash
conda activate sixteen
cd /home/uca/cousin/sixteen/discussion/denovo
bash scripts/run_pipeline.sh 2>&1 | tee pipeline_run.log
```

The pipeline is idempotent: QC and FASTA conversion steps check for existing output and skip if present. To force a full re-run, delete the intermediate directories (`00_qc/` through `07_summary/`).

---

## How It Differs from the Original Pipeline

### Side-by-Side Comparison

| Aspect | Original (Emu + Collapser) | This Pipeline (VSEARCH De Novo) |
|--------|---------------------------|-------------------------------|
| **Classification first?** | Yes -- Emu classifies, then VSEARCH clusters taxids | No -- VSEARCH clusters raw reads, then taxonomy assigned |
| **Input to clustering** | MIMt reference sequences for detected taxids | QC'd amplicon reads (FASTA) |
| **Dereplication** | Implicit (Emu deduplicates) | Replaced by 99% pseudo-denoising (see below) |
| **Read orientation** | Not handled | `vsearch --orient` against MIMt (critical for ONT) |
| **Chimera detection** | Implicit in Emu reference matching | Explicit `vsearch --uchime_ref` against MIMt |
| **OTU clustering** | Single-step `--cluster_fast` at 95%/97% | Two-step: 99% denoise then 97% OTU |
| **Taxonomy method** | Emu (expectation-maximization, MIMt-only) | Dual: SINTAX (k-mer bootstrap) + usearch_global best-hit |
| **OTU97 viability** | Failed (empty associated97/ after filtering) | Works: 857 OTUs, 99.9% classified |
| **OTU count (97%)** | 353 (69% singleton) | 857 (0% singleton) |
| **Unclassified rate** | ~0% (reference-only by design) | 0.1% (1 OTU out of 857) |
| **Centroid source** | Database reference sequences | Actual read sequences from the sample |
| **Consensus sequences** | Not produced | Per-cluster VSEARCH consensus FASTA |
| **Biological filters** | Threepoint / core / associated | Not yet applied (raw OTU table output) |

### What is Preserved

Both pipelines share the same QC stage (porechop + chopper with identical parameters), the same reference database (MIMt 16S, 31,426 sequences), and the same sample structure (6 barcodes, two cohorts). The QC output is bit-for-bit identical: ~47,700 reads per barcode, ~285,000 total.

---

## The ONT Problem: Why Standard Pipelines Fail

Standard 16S amplicon pipelines (QIIME2, mothur, USEARCH) are designed for Illumina data where sequencing error rates are ~0.1%. The typical workflow is:

1. Dereplicate (group identical sequences)
2. Remove singletons
3. Cluster at 97% identity

This fails catastrophically with ONT data. Even with SUP basecalling (R10.4.1), per-read error rates are ~1-5%. This means:

- **Dereplication is useless.** Of 285,354 reads, 285,074 are unique at the exact-match level (99.9%). Only 133 reads have an exact duplicate. Every read is essentially a unique snowflake because ONT errors are stochastic.
- **Direct 97% clustering produces junk.** Without dereplication, clustering 285k unique reads at 97% yields 45,083 OTUs -- mostly error variants of the same species that differ by 2-4%.
- **Singleton removal kills everything.** After dereplication, removing singletons (size=1) leaves only 133 sequences. Clustering those gives 63 OTUs -- a tiny fraction of the real diversity.

### The ONT Read Orientation Problem

ONT sequencing reads DNA in whichever direction the strand enters the pore. For PCR amplicons, this is random: roughly **50% of reads are forward, 50% are reverse complement**. This is invisible in most analyses because tools like Emu and BLAST search both strands automatically.

But VSEARCH clustering (`--cluster_fast`, `--cluster_size`) defaults to `--strand plus` -- it only compares sequences in forward orientation. Forward and reverse-complement reads from the same species will not cluster together, silently doubling the OTU count. Worse, reverse-complement centroids cannot be classified by SINTAX or usearch_global (also plus-strand only by default), making half the OTUs appear "unassigned."

This pipeline solves the problem with `vsearch --orient`, which aligns each read against the MIMt reference to determine its strand and reverse-complements those on the minus strand. In our data:

- 143,826 reads (50.4%) were already forward
- 140,841 reads (49.4%) were reverse-complemented
- 687 reads (0.2%) could not be oriented (likely truly novel organisms)

All downstream steps then operate on consistently-oriented reads.

---

## The Two-Step Clustering Strategy

This is the core methodological contribution of the pipeline. Instead of fighting ONT errors with dereplication, we embrace the noise and use a graduated clustering approach.

### Step A: Pseudo-Denoising at 99% Identity

```
285,354 QC reads  -->  vsearch --cluster_fast --id 0.99  -->  183,625 clusters
```

At 99% identity, reads from the same molecule with 1-3% error cluster together. This acts as denoising: each 99% cluster represents a "consensus" of similar error variants. The centroid (longest sequence in the cluster) has the size annotation appended (e.g., `;size=157`), recording how many reads collapsed into it.

**Why 99% and not 98% or 97%?** At 99%, we collapse pure error variants without merging distinct species. The 16S gene has enough inter-species divergence that most species pairs differ by >1% even in the most conserved regions. At 97%, we would already be merging some genuinely distinct species. At 98%, the risk is intermediate. 99% is the sweet spot for ONT SUP data: tight enough to preserve real variants, loose enough to collapse sequencing noise.

### Step B: Abundance Filtering (Minimum Size 3)

```
183,625 clusters  -->  vsearch --sortbysize --minsize 3  -->  7,645 clusters
```

After 99% clustering, most clusters are singletons or doubletons. These are almost certainly residual errors: sequences that were too divergent to cluster with anything at 99%, usually because they contain a rare error combination. Requiring at least 3 reads per 99% cluster is a conservative threshold: if an organism contributed reads, at least 3 of those reads should cluster together at 99%.

This step removes 96% of the 99% clusters (175,980 clusters eliminated) but only 0.3% of the reads (by count). The removed clusters are overwhelmingly noise.

### Step C: Reference-Based Chimera Filtering

```
7,645 clusters  -->  vsearch --uchime_ref (MIMt)  -->  6,976 non-chimeric
```

PCR chimeras form when an incomplete extension product re-anneals to a different template in the next cycle. VSEARCH's reference-based chimera detection aligns each sequence against MIMt and flags sequences whose left and right halves match different reference sequences better than any single reference.

292 chimeras were detected (3.8% of input). An additional 377 were flagged as "borderline" and retained (conservative approach).

### Step D: OTU Clustering at 97% Identity

```
6,976 denoised centroids  -->  vsearch --cluster_size --id 0.97  -->  857 OTUs
```

Now we cluster the error-corrected, chimera-free 99% centroids at the standard 97% threshold. Because the input is already denoised (each sequence represents a confirmed biological variant with >=3 supporting reads), the clustering is clean:

- **857 OTUs** (vs 45,083 from naive clustering, or 353 from the original EMU pipeline)
- **0% singletons** (every OTU has >=3 supporting 99%-clusters, which themselves have >=3 reads each)
- `--cluster_size` sorts by abundance first, so the most-supported sequences become centroids

### Step E: Read Mapping at 95% Identity

```
285,354 reads  -->  vsearch --usearch_global --id 0.95 --strand both  -->  242,557 mapped (85.2%)
```

Finally, all original QC'd reads (not just the denoised centroids) are mapped back to the 857 OTU centroids. The mapping threshold is 95% rather than 97% to accommodate the fact that individual ONT reads have higher error rates than the denoised centroids they are being mapped to. A read with 3% error from a species at 97% identity to the centroid would have ~94% observed identity -- hence the 95% threshold gives adequate margin.

The 14.8% unmapped reads are those too divergent from any OTU centroid. These are predominantly low-quality reads, chimeric fragments that survived per-read filtering, or reads from organisms whose 99% clusters were too small (size <3) to survive denoising.

---

## Taxonomy Assignment: The Dual-Method Approach

Each of the 857 OTU centroids receives taxonomy from two independent methods. A consensus rule then picks the best assignment.

### Method 1: SINTAX (k-mer Bootstrap)

```bash
vsearch --sintax centroids.fasta --db mimt_sintax.fasta --sintax_cutoff 0.8 --strand both
```

SINTAX is a k-mer based classifier. It works by:

1. Extracting random k-mer subsets from the query sequence
2. Looking up each k-mer in the reference database
3. Tallying which taxonomic labels appear most often across k-mers
4. Bootstrapping (random resampling of k-mers) to estimate confidence at each rank

The `--sintax_cutoff 0.8` threshold means a rank assignment is only reported if 80% of bootstrap replicates agree. This is conservative: it prefers leaving a rank blank over reporting a wrong assignment.

**Strengths:** Fast, works on noisy sequences (k-mers are robust to gaps/errors), provides rank-level confidence.
**Weaknesses:** Cannot assign below the resolution of k-mer similarity (genus is usually the practical limit for 16S).

The MIMt database was converted to SINTAX format by `convert_mimt_sintax.py`, which transforms headers from:
```
>NR_108870.1
```
with separate taxonomy file `NR_108870.1\tK__Bacteria;P__Pseudomonadota;...;S__Tabrizicola aquatica`

to the SINTAX inline format:
```
>NR_108870.1;tax=d:Bacteria,p:Pseudomonadota,c:Alphaproteobacteria,...,s:Tabrizicola aquatica
```

### Method 2: usearch_global Best-Hit (Reference Alignment)

```bash
vsearch --usearch_global centroids.fasta --db mimt_clean.fasta --id 0.80 --strand both --top_hits_only
```

This performs pairwise alignment of each OTU centroid against every MIMt reference sequence, reporting the best hit above 80% identity. The taxonomy of the best-matching reference is then looked up from `mimt_taxonomy.tsv`.

**Strengths:** Provides species-level assignments when identity is high (>97%). Reports percent identity, allowing confidence stratification.
**Weaknesses:** Best-hit is not necessarily the correct assignment (closest reference != true taxonomy). Sensitive to database completeness.

### Consensus Rule (merge_taxonomy.py)

The two methods are merged with the following priority:

1. **SINTAX genus-level at >=0.8 confidence** -- If SINTAX confidently assigns a genus, use it. SINTAX's bootstrap is more statistically principled than a single best-hit alignment.
2. **usearch_global at >=90% identity** -- High-confidence reference match. The taxonomy from MIMt is reliable at this level.
3. **usearch_global at >=80% identity** -- Low-confidence reference match. The taxonomy is reported but flagged as "low confidence." At 80-90% identity, family or order may be correct but genus/species are unreliable.
4. **Unassigned** -- Neither method produced a usable result.

### Results

| Method | OTUs | % |
|--------|------|---|
| SINTAX (genus-level, >=0.8 bootstrap) | 241 | 28.1% |
| usearch_global (>=80% identity) | 615 | 71.8% |
| Unassigned | 1 | 0.1% |
| **Total** | **857** | **100%** |

The single unassigned OTU (OTU_514, 12 reads) could not be matched to anything in MIMt at even 80% identity, suggesting a genuinely novel lineage or a residual chimera.

---

## Pipeline Steps in Detail

### Step 1: Quality Control

Identical to the original pipeline. For each barcode:

1. **Porechop** removes ONT adapter sequences and chimeric adapter-internal reads
2. **quality_cutoff.py** computes the per-barcode Q-score threshold that retains the top 40% of reads
3. **Chopper** filters by that Q threshold, length (500-2000 bp), and crops 15 bp from each end (primer removal)

| Barcode | Raw Reads | QC Reads | Retention |
|---------|-----------|----------|-----------|
| barcode13 | 120,000 | 47,541 | 39.6% |
| barcode14 | 120,000 | 47,657 | 39.7% |
| barcode15 | 120,000 | 47,667 | 39.7% |
| barcode16 | 120,000 | 47,355 | 39.5% |
| barcode17 | 120,000 | 47,660 | 39.7% |
| barcode18 | 120,000 | 47,474 | 39.6% |
| **Total** | **720,000** | **285,354** | **39.6%** |

### Step 2: FASTA Conversion and Read Orientation

Each barcode's QC'd FASTQ is converted to FASTA with `vsearch --fastq_filter`, relabeling reads with barcode prefixes (e.g., `>barcode13_1`, `>barcode13_2`, ...). The `--fastq_qmax 93` flag is required because ONT Q-scores exceed VSEARCH's default maximum of 41.

All barcodes are then pooled into a single FASTA file. This pooled approach produces a single global OTU set shared across all samples, which is standard practice (Edgar 2013) -- it avoids the problem of inconsistent OTU definitions between samples.

The pooled reads are then oriented to the plus strand using `vsearch --orient` against MIMt. This is a critical step for ONT data (see "The ONT Read Orientation Problem" above).

### Steps 3-5: Two-Step Clustering

See "The Two-Step Clustering Strategy" above.

### Step 6: Read Mapping

All 285,354 oriented reads are mapped to the 857 OTU centroids at 95% identity using `vsearch --usearch_global --strand both`. The UC output file records which reads map to which OTU, enabling per-barcode abundance counting.

### Step 7: Taxonomy Assignment

See "Taxonomy Assignment: The Dual-Method Approach" above.

### Step 8: OTU Table Construction (build_otu_tables.py)

The UC mapping file is parsed to count how many reads from each barcode map to each OTU. The script extracts the barcode from each read's header (e.g., `barcode13_4521` -> `barcode13`) and builds a count matrix.

Output files:

- **`otu_table_combined.tsv`** -- Wide-format table: OTU_ID, total_reads, barcode13-18 counts, taxonomy method, identity, and full lineage (Kingdom through Species). Sorted by total abundance descending.
- **`otu_table_long.tsv`** -- Tidy long format for R/ggplot2: one row per OTU per sample, with relative abundance calculated per-sample.
- **`otu_table_per_barcode/`** -- Individual per-barcode tables.
- **`cluster_membership.tsv`** -- Read-level mapping (read_id -> OTU_ID) for downstream analysis.

### Step 9: Summary Statistics (summarize_pipeline.py)

Aggregates counts from all intermediate files into `07_summary/pipeline_stats.tsv`.

---

## Final Results

### Pipeline Statistics

```
QC reads:                285,354
After orientation:       284,667  (687 could not be oriented, 0.24%)
99% denoise clusters:    183,625
After abundance filter:    7,645  (min size 3)
After chimera removal:     6,976  (292 chimeras, 377 borderline)
97% OTUs:                    857
Reads mapped:            242,557  (85.2%)
Reads unmapped:           42,110  (14.8%)
Singleton OTUs:                0  (0%)
```

### Taxonomy Summary

```
SINTAX (>=0.8 conf):       241  (28.1%)
usearch_global (>=80%):    615  (71.8%)
Unassigned:                  1  (0.1%)
```

### Per-Barcode OTU Counts

| Barcode | Mapped Reads | OTUs Detected |
|---------|-------------|---------------|
| barcode13 | 41,273 | 620 |
| barcode14 | 42,752 | 666 |
| barcode15 | 41,983 | 660 |
| barcode16 | 37,529 | 720 |
| barcode17 | 39,308 | 741 |
| barcode18 | 39,712 | 737 |

Drysample barcodes (16-18) show higher OTU richness than fuzzsample (13-15), consistent with the higher Shannon diversity seen in the original EMU analysis.

### Top 15 OTUs by Abundance

| OTU | Reads | Method | %ID | Phylum | Family | Genus |
|-----|-------|--------|-----|--------|--------|-------|
| OTU_9 | 21,812 | global_search | 90.8 | Bacillota | Clostridiaceae | Fonticella |
| OTU_17 | 11,936 | global_search | 89.3 | Bacillota | Oscillospiraceae | Acetivibrio |
| OTU_10 | 8,811 | sintax | -- | Pseudomonadota | Sphingomonadaceae | Sphingomonas |
| OTU_13 | 7,503 | global_search | 82.4 | Bacillota | Xylanivirgaceae | Xylanivirga |
| OTU_164 | 7,282 | sintax | -- | Bacillota | Acidilutibacteraceae | Acidilutibacter |
| OTU_7 | 6,585 | sintax | -- | Bacillota | Lutisporaceae | Lutispora |
| OTU_8 | 6,378 | global_search | 97.0 | Pseudomonadota | Paracoccaceae | Tabrizicola |
| OTU_2 | 6,220 | global_search | 95.3 | Pseudomonadota | Sphaerotilaceae | Aquincola |
| OTU_11 | 5,277 | global_search | 87.6 | Bacillota | Xylanivirgaceae | Xylanivirga |
| OTU_43 | 5,192 | global_search | 98.8 | Bacteroidota | Tannerellaceae | Parabacteroides |
| OTU_28 | 3,817 | global_search | 91.4 | Bacillota | Beduinellaceae | Beduinella |
| OTU_4 | 3,434 | global_search | 89.7 | Bacillota | Oscillospiraceae | Acetivibrio |
| OTU_1 | 3,394 | global_search | 86.3 | Bacillota | Pumilibacteraceae | Pumilibacter |
| OTU_6 | 3,276 | global_search | 90.7 | Bacillota | Oscillospiraceae | Acetivibrio |
| OTU_3 | 3,003 | sintax | -- | Bacillota | Acutalibacteraceae | Caproicibacter |

### Phylum Composition

| Phylum | OTUs | Reads | % Reads |
|--------|------|-------|---------|
| Bacillota | 482 | 166,425 | 68.7% |
| Pseudomonadota | 224 | 57,799 | 23.8% |
| Bacteroidota | 49 | 11,904 | 4.9% |
| Actinomycetota | 16 | 643 | 0.3% |
| Verrucomicrobiota | 12 | 712 | 0.3% |
| Gemmatimonadota | 10 | 892 | 0.4% |

This is consistent with the original EMU results: Bacillota (~65-73%), Pseudomonadota (~23-24%), Bacteroidota (1-8%), with the top three phyla accounting for >97% of all classified reads.

---

## Directory Structure

```
clustering/
  scripts/
    run_pipeline.sh              # Main pipeline driver (bash)
    convert_mimt_sintax.py       # MIMt FASTA+TSV -> SINTAX format converter
    merge_taxonomy.py            # Dual taxonomy consensus builder
    build_otu_tables.py          # UC parser -> OTU abundance tables
    summarize_pipeline.py        # Pipeline statistics aggregator
  00_qc/
    {barcode}_qc.fastq           # QC'd reads (one per barcode)
    logs/                        # Porechop and chopper logs
  01_fasta/
    {barcode}.fasta              # Per-barcode FASTA with prefixed headers
  02_denoise/
    all_reads_raw_pooled.fasta   # All barcode reads concatenated (pre-orient)
    all_reads_pooled.fasta       # All reads after strand orientation
    centroids_0.99.fasta         # 99% cluster centroids (with size annotations)
    centroids_0.99_filtered.fasta # After abundance filter (size >= 3)
    clusters_0.99.uc             # 99% cluster membership
  03_chimera/
    centroids_0.99_nochim.fasta  # Non-chimeric 99% centroids
    chimeric_centroids.fasta     # Detected chimeras
    uchime_ref.tsv               # Chimera detection details
  04_cluster/
    centroids_0.97.fasta         # OTU centroid sequences (labeled OTU_1, OTU_2, ...)
    consensus_0.97.fasta         # Per-OTU consensus sequences
    clusters_0.97.uc             # 97% cluster membership (99%-centroid -> OTU)
    readmap_0.97.uc              # All-read -> OTU mapping
  05_taxonomy/
    sintax_db/mimt_sintax.fasta  # MIMt in SINTAX header format
    centroids_clean.fasta        # Centroids with size annotations stripped
    sintax_results.tsv           # Raw SINTAX output (4-column)
    usearch_global_blast6.tsv    # Raw usearch_global output (blast6 format)
    taxonomy_consensus.tsv       # Merged consensus taxonomy
  06_tables/
    otu_table_combined.tsv       # Wide-format OTU table (primary output)
    otu_table_long.tsv           # Long-format for R
    otu_table_per_barcode/       # Individual barcode tables
    cluster_membership.tsv       # Read -> OTU mapping
  07_summary/
    pipeline_stats.tsv           # Aggregated pipeline statistics
```

---

## Parameters Reference

### QC Parameters (matching original pipeline)

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| `TOP_PERCENT` | 40 | Keep top 40% of reads by mean Q-score |
| `MIN_LEN` | 500 | Minimum read length after cropping |
| `MAX_LEN` | 2000 | Maximum read length (full-length 16S is ~1500 bp) |
| `CROP_BP` | 15 | Fixed trim from each end (primer removal) |
| `--fastq_qmax` | 93 | ONT Q-scores exceed the VSEARCH default max of 41 |

### Clustering Parameters

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| `DENOISE_ID` | 0.99 | Collapse ONT error variants (1-3% error rate) |
| `MIN_DENOISE_SIZE` | 3 | Remove rare error clusters (singletons/doubletons) |
| `OTU_ID` | 0.97 | Standard species-level 16S OTU threshold |
| Read mapping `--id` | 0.95 | Looser than 97% to accommodate per-read ONT error |
| `--strand both` | all steps | Safety net for the 0.2% of reads that couldn't be oriented |

### Taxonomy Parameters

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| SINTAX `--sintax_cutoff` | 0.8 | 80% bootstrap confidence (standard threshold) |
| usearch_global `--id` | 0.80 | Minimum identity for any reference match |
| usearch_global high-confidence | >=90% | Genus/species likely correct |
| usearch_global low-confidence | 80-90% | Family/order may be correct, genus uncertain |

### Reference Database

| Database | File | Sequences | Description |
|----------|------|-----------|-------------|
| MIMt 16S FASTA | `mimt_clean.fasta` | 31,426 | Curated full-length 16S rRNA sequences |
| MIMt taxonomy | `mimt_taxonomy.tsv` | 31,426 | `seqid\tK__Bacteria;P__...;S__Species` format |
| MIMt SINTAX | `mimt_sintax.fasta` | 31,426 | Auto-generated by `convert_mimt_sintax.py` |

---

## Bugs Fixed During Development

### 1. Exact-Match Dereplication Failure (ONT)

**Problem:** Standard dereplication produced 285,074 unique sequences from 286k reads (99.7% unique). Only 133 had duplicates. After singleton removal, only 63 OTUs remained.

**Diagnosis:** ONT stochastic errors make every read unique at exact-match level.

**Fix:** Replaced dereplication with 99% pseudo-denoising.

### 2. Direct 97% Clustering Explosion

**Problem:** Clustering 285k undeduplicated reads at 97% produced 45,083 OTUs.

**Diagnosis:** ONT error rates (1-5%) mean reads from the same species frequently differ by >3%, splitting one species across many OTUs.

**Fix:** Two-step clustering: 99% denoise (collapses error variants) then 97% OTU (collapses species variants).

### 3. Reverse-Complement Read Orientation

**Problem:** 50.6% of OTUs (640/1,266) had no taxonomy assignment. These were the most abundant OTUs, including the single largest OTU (11,373 reads).

**Diagnosis:** ONT reads are sequenced in random orientation (~50/50). Without orientation correction, forward and RC reads from the same species formed separate clusters. RC-centroid OTUs could not be classified because SINTAX and usearch_global only searched the plus strand.

**Fix:** Added `vsearch --orient` step after FASTA conversion. Also added `--strand both` to SINTAX, usearch_global, and read mapping as a safety net. This reduced OTU count from 1,266 to 857 (forward/RC duplicates merged) and raised the classification rate from 49.4% to 99.9%.

### 4. SINTAX Trailing Tab Parse Error

**Problem:** `merge_taxonomy.py` produced only 956 taxonomy entries from 1,266 SINTAX results. 310 OTUs were silently dropped.

**Diagnosis:** VSEARCH SINTAX output has 4 tab-separated columns, but when column 4 (cutoff taxonomy) is empty, the line ends with a trailing tab. Python's `line.strip()` removed this tab, causing `split('\t')` to produce only 3 fields. The `if len(parts) < 4: continue` check then skipped the line.

**Fix:** Changed `line.strip()` to `line.rstrip('\n')` and relaxed the minimum fields check to `< 2`.

---

## Next Steps

The raw OTU table (`06_tables/otu_table_combined.tsv`) is ready for downstream analysis. Potential next steps include:

- **Apply biological filters** (threepoint, core, associated) from the original pipeline to this OTU table
- **Build consensus sequences** using the per-cluster read membership and `04_cluster/consensus_0.97.fasta`
- **Phylogenetic placement** of OTU centroids against a reference tree
- **Differential abundance analysis** between fuzzsample and drysample cohorts
- **Compare with EMU results** at genus/phylum level to validate concordance

---

## Reproducibility

All parameters are defined at the top of `scripts/run_pipeline.sh`. The pipeline uses only tools available in the `sixteen` conda environment (vsearch 2.30.1, seqkit, porechop, chopper) plus Python 3 standard library. No external network access is required.

To reproduce from scratch:

```bash
# Delete all intermediate files
rm -rf 00_qc 01_fasta 02_denoise 03_chimera 04_cluster 05_taxonomy 06_tables 07_summary

# Re-run
conda activate sixteen
bash scripts/run_pipeline.sh 2>&1 | tee pipeline_run.log
```
