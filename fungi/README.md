# funcall (fungal ITS/18S pipeline)

Comprehensive fungal metabarcoding pipeline for Oxford Nanopore ITS/18S amplicon sequencing, spanning basecalling through diversity analysis and co-occurrence networks.

## Project Context

This pipeline analyzes fungal communities from soil samples collected at the **anonymized contaminated site** in New Zealand—a location historically contaminated with 2,3,7,8-Tetrachlorodibenzo-p-dioxin (TCDD) from industrial production. The analysis aims to characterize:

- **Fungal community structure** in dioxin-contaminated soils
- **Detection of POP-degrader associated taxa** (white-rot fungi, xenobiotic metabolizers)
- **Plant-beneficial symbionts** (ectomycorrhizal and arbuscular mycorrhizal fungi)
- **Soil health indicators** and pollution-tolerant guilds
- **Co-occurrence networks** for microbial interaction patterns

**Sample barcodes:**
- barcode19, barcode20, barcode21: Fungal ITS/18S amplicons (n=3)
- unclassified: Mixed/ambiguous barcode assignments

**Related workflows:**
- Bacterial 16S analysis: [sixteen/README.md](../sixteen/README.md)
- Targeted gene analysis: [roadtrip/README.md](../roadtrip/README.md)
- Thesis writing: [waffle/README.md](../waffle/README.md)

---

## Installation & Requirements

### Hardware Requirements
- **GPU:** CUDA-capable (tested: NVIDIA RTX 3060 Ti with CUDA 11+)
- **RAM:** 32GB minimum (EMU classification and clustering stages)
- **Disk:** 100GB for intermediates and outputs
- **Note:** CPU-only basecalling is 20× slower

### Software Prerequisites
- Mamba/Conda (package management)
- Nextflow ≥23.0 (workflow execution)
- Java 17 (Nextflow dependency)
- CUDA drivers (GPU basecalling)

### Environment Setup
```bash
# Create environment from comprehensive spec
mamba env create -f sixtran/funcall.yml
conda activate funcall

# Verify database and tool symlinks
ls -l house/input          # → data/raw/funcall/house/input
ls -l mapping/refdbs       # → data/refs/funcall/mapping/refdbs
ls -l dordowna             # → data/tools/dordowna
```

### Database Locations (gitignored)
Actual storage under `data/`, symlinked into pipeline paths:
- [house/input](house/input) → [data/raw/funcall/house/input](../data/raw/funcall/house/input) (POD5 files)
- [mapping/refdbs](mapping/refdbs) → [data/refs/funcall/mapping/refdbs](../data/refs/funcall/mapping/refdbs) (EMU databases)
- [dordowna](dordowna) → [data/tools/dordowna](../data/tools/dordowna) (Dorado basecaller)

If `data/` moves to a new drive/mount, update these symlinks. See workspace root [README.md](../README.md) for data management policy.

---

## Quick Start Overview

The pipeline follows this structure from raw reads to final diversity analyses:

```
POD5 files (848K reads)
    ↓ [Basecalling: Dorado HAC]
FASTQ (849K reads)
    ↓ [Demultiplexing: retain basecaller barcodes]
Barcoded FASTQs (bc19=102K, bc20=248K, bc21=434K)
    ↓ [QC filtering: Q≥15, 1-7kb, adapter trim]
Filtered FASTQs (360K reads, ~42% retention)
    ↓ [Taxonomic classification: EMU]
Abundance tables (1,306 taxa detected)
    ↓ [Fungal filtering: phylum-level]
Fungi-only (876 entries, 67% of total)
    ↓ [OTU clustering: Vsearch 95%]
OTU tables (Parrot: 411 OTUs | Pukeko: 83 fungal OTUs)
    ↓ [Diversity & networks]
Alpha/beta diversity, CLR networks, target pull-downs
```

**Three main output variants:**
- **Parrot:** All-taxa OTU clustering (607 taxids → 411 OTUs)
- **Pukeko:** Fungi-only refined (241 taxa → 83 OTUs, 60% singletons)
- **Snipe:** Target pull-down (15/51 genera detected from curated list)

For conceptual overview, see [sixtran/primer.md](sixtran/primer.md). For current pipeline state, see [sixtran/where-we-are.md](sixtran/where-we-are.md).

---

## Pipeline Stages

### Stage 1: Basecalling

**Directory:** [house/](house/)  
**Tool:** Dorado 1.3.0 with HAC (high-accuracy) model  
**Model:** `dna_r10.4.1_e8.2_400bps_hac@v4.3.0`  
**Hardware:** GPU-accelerated (CPU mode 20× slower)

**Input:**
- POD5 files in [house/input](house/input) (symlinked to [data/raw/funcall/house/input](../data/raw/funcall/house/input))
- Sequencing run: `20230303_1429_MC-115797_FAX53729_8bbd279f`

**Process:**
```bash
# Example basecalling command (see house/admin/ for dated logs)
dordowna/bin/dorado basecaller \
    dna_r10.4.1_e8.2_400bps_hac@v4.3.0 \
    house/input/Fungi/.../pod5_pass/ \
    --emit-fastq > house/output/basecalled.fastq
```

**Metrics:**
- Reads basecalled: 848,367 simplex reads
- Failed QC: 31 reads (0.004% failure rate)
- Runtime: ~41 minutes on RTX 3060 Ti
- Mean read length: 5.0-5.6 kb (ITS + 18S amplicons)

**Output:** [house/output/noclass/fastqpass/](house/output/noclass/fastqpass/)

**See also:** [house/README.md](house/README.md) for detailed commands and admin logs in [house/admin/](house/admin/).

---

### Stage 2: Demultiplexing

**Directory:** [house/](house/)  
**Kit:** SQK-NBD114-24 (24-plex native barcoding)  
**Strategy:** Retain basecaller barcode assignments (`--no-classify --no-trim`)

**Rationale:** Initial kit-based demultiplexing yielded 99.7% unclassified reads. Retaining basecaller assignments during basecalling reduced unclassified to 7.7%.

**Metrics (basecaller-assigned demux):**
| Barcode | Reads | Mean Length | Median Length |
|---------|-------|-------------|---------------|
| barcode19 | 101,944 | 5,589 bp | 5,166 bp |
| barcode20 | 248,391 | 5,316 bp | 5,177 bp |
| barcode21 | 433,973 | 4,998 bp | 5,166 bp |
| unclassified | 65,375 | 5,202 bp | 5,243 bp |
| **Total** | **849,683** | **5,165 bp** | **5,172 bp** |

All reads: 0% N bases, high basecall quality retained.

**Output:** Per-barcode FASTQs in [house/output/noclass/fastqpass/](house/output/noclass/fastqpass/)

---

### Stage 3: Quality Filtering

**Directory:** [oven/slashing/](oven/slashing/)  
**Tools:** Porechop (adapter trimming) + Chopper (quality/length filtering)  
**Configuration:** Morris round 2 (refined parameters)

**Parameters:**
- **MINQ:** 15 (Phred quality score)
- **MINLEN:** 1000 bp (ITS lower bound)
- **MAXLEN:** 7000 bp (accommodates full 18S+ITS, increased from initial 3500 bp)
- **HEADCROP:** 15 bp (remove adapter remnants)
- **TAILCROP:** 15 bp

**Retention Metrics (Morris round 2):**
| Barcode | Input Reads | Output Reads | Retention | Avg Length (bp) |
|---------|-------------|--------------|-----------|-----------------|
| barcode19 | 101,944 | 43,043 | **42.2%** | 5,589 → 4,971 |
| barcode20 | 248,391 | 104,959 | **42.3%** | 5,316 → 4,965 |
| barcode21 | 433,973 | 192,864 | **44.4%** | 4,998 → 5,034 |
| unclassified | 65,375 | 18,978 | **29.0%** | 5,202 → 5,032 |
| **Overall** | **849,683** | **359,844** | **~42%** | **5,165 → 5,003** |

**Rationale:**
- Q≥15 improves EMU mapping accuracy vs Q≥10 initial
- MAXLEN=7000 captures full-length amplicons without truncating
- ~42% retention acceptable for long amplicons with adapter trimming

**Output:** Filtered FASTQs in [oven/slashing/morris/filtered/](oven/slashing/morris/filtered/)  
**Source:** [oven/slashing/morris/summary.tsv](oven/slashing/morris/summary.tsv)

**See also:** [oven/slashing/README.md](oven/slashing/README.md) for filter development history.

---

### Stage 4: Taxonomic Classification

**Directory:** [mapping/](mapping/)  
**Tool:** EMU (Expectation-Maximization classifier optimized for ONT reads)  
**Workflow:** Nextflow DSL2 pipeline ([sixtran/main.nf](sixtran/main.nf))

**Database (primary):** MIMt 18S+ITS combined
- Location: [mapping/refdbs/mergemimt/emu_db](mapping/refdbs/mergemimt/emu_db)
- Sequences: 61,924 reference sequences (fungal-focused)
- Taxonomy: NCBI taxdump
- ID prefixes: `MIMT18S|`, `MIMTITS|` (avoid namespace collisions)

**Alternative database:** UNITE ITS general release
- Location: [mapping/refdbs/unit/emu_db](mapping/refdbs/unit/emu_db)
- Sequences: 100,176 (broader coverage, less curated)

**Process:**
```bash
# Per-sample EMU classification
emu map-ont --keep-counts --keep-files \
    --db mapping/refdbs/mergemimt/emu_db \
    oven/slashing/morris/filtered/barcode19.fastq

# Combine tables with taxonomy
python sixtran/bin/merge_emu_tables.py
```

**Classification Results:**
- **Total taxa detected:** 1,306 unique taxa across all samples
- **Fungal signal:** 876 entries (67.1% of total)
- **Non-fungal contaminants:** 429 entries (32.9%)
  - Plants (Streptophyta): 129 entries
  - Animals (Arthropoda, Chordata): ~195 entries
  - Algae (Chlorophyta): 12 entries
  - Oomycetes: 20 entries

**Per-sample diversity:**
| Barcode | Fungal Species | Fungal Genera |
|---------|----------------|---------------|
| barcode19 | 135 | 107 |
| barcode20 | 282 | 206 |
| barcode21 | 375 | 244 |
| unclassified | 84 | 72 |

**Output:** [mapping/out/emu_slash/emu_abundance.tsv](mapping/out/emu_slash/emu_abundance.tsv)

**Columns:** sample_id, tax_id, abundance, species, genus, family, order, class, phylum, superkingdom, estimated_counts

**See also:** [mapping/README.md](mapping/README.md) for database build instructions and EMU parameters.

---

### Stage 5: OTU Clustering

**Directories:** [parrot/](parrot/) (all-taxa), [pukeko/](pukeko/) (fungi-only)  
**Tool:** Vsearch 2.12.0 `--cluster_fast`  
**Identity threshold:** 95% (approximately genus-level resolution)  
**Clustering mode:** Comparative (shared OTU IDs across all samples for beta diversity)

**Preprocessing:**
1. Build collapser tables from EMU results ([scripts/collapser_build.py](scripts/collapser_build.py))
2. Apply read threshold: ≥10 estimated reads (analysis grade) or ≥1 (trace sensitivity)
3. Extract representative sequences per taxid from reference database
4. Map taxids to sequence IDs via [mapping/refdbs/mergemimt/mimt_mapping.fixed.tsv](mapping/refdbs/mergemimt/mimt_mapping.fixed.tsv)

**Clustering process:**
```bash
# Extract representatives and cluster (see scripts/collapser_otu95.sh)
vsearch --cluster_fast representatives.fasta \
    --id 0.95 \
    --centroids centroids_0.95.fasta \
    --uc clusters_0.95.uc

# Aggregate abundances by cluster
python scripts/collapser_cluster_memberships.py
```

#### Parrot Workflow (All-Taxa)

**Input:** 607 unique taxids (all superkingdoms)  
**Output:** 411 OTU clusters  
**Compression ratio:** 1.48:1 (taxa-to-OTU)

**Per-sample OTU counts:**
- barcode19: 185 OTUs
- barcode20: 272 OTUs
- barcode21: 334 OTUs
- unclassified: 115 OTUs

**Files:**
- OTU table: [parrot/rout/collapser/otu95/collapser_abundance_0.95.tsv](parrot/rout/collapser/otu95/collapser_abundance_0.95.tsv)
- Centroids: [parrot/rout/collapser/otu95/centroids_0.95.fasta](parrot/rout/collapser/otu95/centroids_0.95.fasta)
- Composition checker: [parrot/rout/collapser/otu95/composition_checker_0.95.tsv](parrot/rout/collapser/otu95/composition_checker_0.95.tsv)
- Taxid mapping: [parrot/rout/collapser/otu95/taxid2cluster.tsv](parrot/rout/collapser/otu95/taxid2cluster.tsv)

**See also:** [parrot/README.md](parrot/README.md) for workflow details and component scripts in [parrot/components/](parrot/components/).

#### Pukeko Workflow (Fungi-Only)

**Preprocessing:** Phylum-level fungal filtering (see Stage 6)  
**Input:** 241 fungal taxa (120 with representative sequences)  
**Output:** 83 OTU clusters  
**Compression ratio:** 1.45:1 (taxa-to-OTU)  
**Singleton proportion:** 60% (OTUs with single taxid member)

**Estimated fungal read counts:** 149,285 total across samples

**Files:**
- OTU table: [pukeko/rout/collapser/otu95/collapser_abundance_0.95.tsv](pukeko/rout/collapser/otu95/collapser_abundance_0.95.tsv)
- Centroids: [pukeko/rout/collapser/otu95/centroids_0.95.fasta](pukeko/rout/collapser/otu95/centroids_0.95.fasta)

**See also:** [pukeko/README.md](pukeko/README.md) for fungi-only refinement rationale and scripts in [pukeko/components/](pukeko/components/).

**Clustering rationale:**
- **95% identity:** More stable than 97% (species-level) for noisy ONT reads; reduces error-driven spurious OTUs
- **Comparative mode:** Essential for beta diversity and network analyses; alternative per-sample mode in [scripts/collapser_otu95_per_sample.sh](scripts/collapser_otu95_per_sample.sh)
- **Centroid selection:** Most abundant sequence per cluster becomes representative

---

### Stage 6: Fungal Filtering

**Directories:** [oven/shroom/](oven/shroom/), [pukeko/](pukeko/)  
**Rationale:** Remove non-fungal contamination (32.9% of EMU hits)

**Method:** Phylum-level retention of true fungi (phylogenetically sound)

**Retained phyla (9 fungal groups):**
1. **Ascomycota** (sac fungi): 377 entries (43%)
2. **Basidiomycota** (club fungi): 255 entries (29%)
3. **Chytridiomycota** (chytrids): 116 entries (13%)
4. **Mucoromycota** (incl. AMF): 85 entries (10%)
5. **Zoopagomycota**: 30 entries (3%)
6. **Cryptomycota**: 6 entries
7. **Microsporidia**: 4 entries
8. **Blastocladiomycota**: 3 entries
9. **Olpidiomycota**: minimal

**Excluded non-fungal groups (429 entries):**
- Plants (Streptophyta): 129 entries
- Animals (Arthropoda, Chordata, Nematoda): ~195 entries
- Algae (Chlorophyta): 12 entries
- Oomycetes (water molds, phylogenetically distinct): 20 entries

**Results:**
- Input: 1,305 total entries
- Retained: 876 fungal entries (**67.1%**)
- Unique species: 449 fungal species

**Example abundant taxa (from [pukeko/emu_abundance_fungal.tsv](pukeko/emu_abundance_fungal.tsv)):**
- *Kwoniella shivajii* (Basidiomycota): 4.6-5.0% relative abundance
- *Chytridium olla* (Chytridiomycota): 1.2-2.9%
- *Mitosporidium daphniae* (Microsporidia): 1.3-1.9%
- *Gonapodya prolifera* (Chytridiomycota): 1.0%
- *Spizellomyces punctatus* (Chytridiomycota): 0.9%

**Functional guilds detected:**
- **Arbuscular mycorrhizal fungi (AMF):** Diversispora, Acaulospora, Jimgerdemannia (Glomeromycetes)
- **Ectomycorrhizal (ECM):** Pisolithus
- **Decomposers/saprotrophs:** Dominated by Ascomycota
- **Animal parasites:** Zoopagomycota (Kickxellales)

**Output:** [oven/shroom/shroom_abundance.tsv](oven/shroom/shroom_abundance.tsv)

**See also:** [oven/shroom/README.md](oven/shroom/README.md) for filtering scripts and non-OTU species-level analyses.

---

### Stage 7: Target Pull-down

**Directory:** [oven/snipe/](oven/snipe/)  
**Purpose:** Extract taxa matching curated functional gene list

**Target list:** 51 fungal genera in 6 categories  
**Source:** [oven/snipe/targetlist.tsv](oven/snipe/targetlist.tsv)

**Categories and example genera:**

| Category | Count | Example Genera | Ecological Role |
|----------|-------|----------------|-----------------|
| POP_bioremediation | 15 | Phanerochaete, Trametes, Pleurotus | White-rot fungi, lignin peroxidases |
| Plant_beneficial_symbiont | 14 | Pisolithus, Rhizophagus, Claroideoglomus | ECM and AM fungi |
| Soil_health_core | 11 | Mortierella, Penicillium, Trichoderma | Ubiquitous saprotrophs |
| Biocontrol_beneficial | 5 | Beauveria, Metarhizium, Purpureocillium | Entomopathogenic fungi |
| Pollution_tolerant | 4 | Exophiala, Cladophialophora | Melanized/black yeasts |
| Pathogen_indicator | 2 | Fusarium, Alternaria | Plant pathogen markers |

**Detection Results:**

| Barcode | Genera Detected | Taxa Screened |
|---------|-----------------|---------------|
| barcode19 | 1 | 120 |
| barcode20 | 3 | 149 |
| barcode21 | 7 | 196 |
| unclassified | 0 | 100 |
| **Combined** | **15/51 (29.4%)** | **565** |

**Key detected genera:**
- Pisolithus (ECM symbiont)
- White-rot fungi (lignin degraders)
- AMF genera (Diversispora, Acaulospora)
- Soil saprotrophs (subset of Soil_health_core)

**Output:**
- Target abundance table: [oven/snipe/target/emu_abundance.tsv](oven/snipe/target/emu_abundance.tsv)
- Detection summary: [oven/snipe/target/summary.tsv](oven/snipe/target/summary.tsv)

**See also:** [oven/snipe/README.md](oven/snipe/README.md) for target list curation rationale and pull-down scripts.

---

### Stage 8: Diversity Analysis & Networks

**Directories:** [networkan/morris/](networkan/morris/), [networkan/snipemorris/](networkan/snipemorris/)

#### Alpha Diversity
**Metrics:** Shannon, Simpson, Chao1, observed OTUs  
**Scripts:** [parrot/components/metrics/](parrot/components/metrics/), [pukeko/components/metrics/](pukeko/components/metrics/)

**Example results (Pukeko fungi-only):**
- barcode21: 70 observed OTUs, Shannon H'=3.51 (highest diversity)
- barcode20: 57 OTUs, Shannon H'=3.22
- barcode19: 41 OTUs, Shannon H'=2.89

**Output:** [parrot/components/metrics/outputs/](parrot/components/metrics/outputs/)

#### Beta Diversity
**Method:** Bray-Curtis dissimilarity, PCoA ordination  
**Purpose:** Visualize community composition differences between barcodes

#### CLR Transformation
**Tool:** [mapping/morris/inference/clr_transform.py](mapping/morris/inference/clr_transform.py)  
**Rationale:** Sequencing counts are compositional data; raw correlations are spurious  
**Method:** Centered log-ratio (CLR) transformation to address compositional constraints

**Output:** [networkan/morris/clr/clr_matrix.tsv](networkan/morris/clr/clr_matrix.tsv)

#### Co-occurrence Networks
**Tool:** [mapping/morris/inference/build_cooccurrence.py](mapping/morris/inference/build_cooccurrence.py)  
**Method:** Pearson/Spearman correlations on CLR-transformed abundances  
**Thresholds:** |r| ≥ 0.60 (primary, conservative for n=4 samples), alternatives tested at 0.50, 0.55, 0.65

**Filters:**
- Minimum shared samples
- Prevalence checks
- Taxa retained post-QC

**Outputs:**
- Network edges: [networkan/morris/networks/](networkan/morris/networks/)
- Network nodes: Taxon metadata with network statistics
- Heatmaps: [networkan/morris/heatmaps/](networkan/morris/heatmaps/)
- Snipe-filtered networks: [networkan/snipemorris/](networkan/snipemorris/)

**Caveats:**
- **Exploratory analysis only:** Small sample size (n=3-4) limits statistical power
- **Correlational, not causal:** Networks represent co-occurrence patterns, not mechanistic interactions
- **Genus labels ≠ functional capacity:** Taxonomic assignment does not guarantee degrader phenotype

**See also:** [mapping/morris/inference/README.md](mapping/morris/inference/README.md) (if exists) for network inference methodology.

---

## Output Files Reference

### Primary Outputs

| File Path | Description | Format | Key Columns |
|-----------|-------------|--------|-------------|
| [house/output/noclass/fastqpass/*.fastq](house/output/noclass/fastqpass/) | Basecalled, demultiplexed reads | FASTQ | Per-barcode files |
| [oven/slashing/morris/filtered/*.fastq](oven/slashing/morris/filtered/) | QC-filtered reads (Q≥15, 1-7kb) | FASTQ | Per-barcode files |
| [mapping/out/emu_slash/emu_abundance.tsv](mapping/out/emu_slash/emu_abundance.tsv) | Combined EMU classification | TSV | sample_id, tax_id, abundance, taxonomy ranks, estimated_counts |
| [parrot/rout/collapser/otu95/collapser_abundance_0.95.tsv](parrot/rout/collapser/otu95/collapser_abundance_0.95.tsv) | All-taxa OTU table (411 OTUs) | TSV | OTU_ID, read_count, estimated_counts, member_taxids, taxonomy |
| [pukeko/rout/collapser/otu95/collapser_abundance_0.95.tsv](pukeko/rout/collapser/otu95/collapser_abundance_0.95.tsv) | Fungi-only OTU table (83 OTUs) | TSV | OTU_ID, read_count, estimated_counts, member_taxids, taxonomy |
| [oven/shroom/shroom_abundance.tsv](oven/shroom/shroom_abundance.tsv) | Fungi-only species-level (449 taxa, no clustering) | TSV | tax_id, abundance, taxonomy, estimated_counts |
| [oven/snipe/target/emu_abundance.tsv](oven/snipe/target/emu_abundance.tsv) | Target genera pull-down (15 genera detected) | TSV | sample_id, tax_id, abundance, taxonomy |
| [parrot/rout/collapser/otu95/centroids_0.95.fasta](parrot/rout/collapser/otu95/centroids_0.95.fasta) | Representative sequences (411 OTU centroids) | FASTA | Headers: OTU_ID, member taxonomy |
| [networkan/morris/clr/clr_matrix.tsv](networkan/morris/clr/clr_matrix.tsv) | CLR-transformed abundances | TSV | Taxa × samples matrix |
| [networkan/morris/networks/](networkan/morris/networks/) | Co-occurrence network edges/nodes | TSV | Correlation coefficients, p-values, node metadata |

### Quality Control Outputs

| File Path | Description |
|-----------|-------------|
| [oven/slashing/morris/summary.tsv](oven/slashing/morris/summary.tsv) | QC filtering retention statistics |
| [parrot/rout/collapser/otu95/composition_checker_0.95.tsv](parrot/rout/collapser/otu95/composition_checker_0.95.tsv) | OTU membership view (which taxa collapsed into each cluster) |
| [parrot/rout/collapser/otu95/taxid2cluster.tsv](parrot/rout/collapser/otu95/taxid2cluster.tsv) | Mapping: taxid → OTU_ID |
| [parrot/rout/collapser/otu95/seq2cluster.tsv](parrot/rout/collapser/otu95/seq2cluster.tsv) | Mapping: sequence_ID → OTU_ID |
| [parrot/components/metrics/outputs/](parrot/components/metrics/outputs/) | Alpha/beta diversity metrics, ordination plots |

### Intermediate Files

| File Path | Description |
|-----------|-------------|
| [parrot/rout/collapser/collapser_abundance.tsv](parrot/rout/collapser/collapser_abundance.tsv) | Pre-clustering abundance table (≥10 reads) |
| [parrot/rout/collapser_trace/collapser_abundance.tsv](parrot/rout/collapser_trace/collapser_abundance.tsv) | Trace sensitivity table (≥1 read) |
| [pukeko/emu_abundance_fungal.tsv](pukeko/emu_abundance_fungal.tsv) | Phylum-filtered fungal taxa before clustering |

---

## Key Decisions & Rationale

### 95% Clustering Identity
**Decision:** Use 95% identity threshold (approximately genus-level)  
**Rationale:**
- More stable than 97% (species-level) for Oxford Nanopore reads with higher error rates
- Reduces spurious OTUs caused by sequencing errors or intra-species variation
- Genus-level resolution sufficient for community ecology and network analyses
- Alternative 97% clustering used in sixteen pipeline for comparison

**Trade-off:** May collapse closely related species; appropriate for exploratory metabarcoding

### Q≥15 Quality Threshold
**Decision:** MINQ=15 (Phred) for QC filtering (Morris round 2)  
**Rationale:**
- Higher than initial Q≥10 to improve EMU mapping accuracy
- Balances retention (~42%) with quality for long amplicons (1-7 kb)
- Oxford Nanopore reads have position-dependent error profiles; Q≥15 filters poor-quality segments

**Impact:** 358K reads retained from 850K raw (42% overall)

### MAXLEN=7000 bp
**Decision:** Extend max length from 3500 bp (Morris round 1) to 7000 bp (round 2)  
**Rationale:**
- Full-length ITS+18S amplicons can exceed 5 kb
- Initial 3500 bp cutoff truncated legitimate long reads
- Read length distribution shows median ~5.2 kb, with valid reads up to 6.5 kb

**Impact:** Improved representation of complete rRNA operon sequences

### Fungal-Only Filtering (Phylum-Level)
**Decision:** Exclude entire non-fungal phyla rather than species-by-species  
**Rationale:**
- 32.9% of EMU hits were plants, animals, algae, oomycetes (environmental DNA contamination)
- Phylum-level filtering is phylogenetically sound; entire lineages are non-fungal
- Avoids subjective case-by-case decisions

**Retained:** 9 fungal phyla (Ascomycota through Olpidiomycota)  
**Excluded:** Oomycota (water molds, distinct lineage), all Streptophyta, Arthropoda, Chordata, etc.

**Impact:** 876/1,305 entries retained (67.1%), improved ecological interpretability

### Read Threshold ≥10
**Decision:** Use ≥10 estimated reads for primary analyses (analysis grade)  
**Rationale:**
- Tested thresholds: 1, 2, 5, 10 estimated reads
- ≥10 provides best signal-to-noise for statistical inference
- Stabilizes against sequencing noise and reduces false positives
- Alternative ≥1 threshold (trace mode) available for comprehensive inventories

**Impact:** Removes most singletons/doubletons; retains 241 fungal taxa

### EMU Database: MIMt 18S+ITS
**Decision:** Use MIMt combined database (61,924 sequences) as primary  
**Rationale:**
- Fungal-focused curation balances coverage and accuracy
- NCBI taxonomy integration for consistent taxid mapping
- Alternative UNITE (100,176 sequences) tested but less curated
- ID prefixes (MIMT18S|, MIMTITS|) prevent namespace collisions

**Trade-off:** Lower sequence count than UNITE but higher curation quality

### Comparative (Shared) OTU Clustering
**Decision:** Cluster all samples together to produce shared OTU IDs  
**Rationale:**
- Essential for beta diversity comparisons and network inference
- Ensures consistent OTU definitions across barcodes
- Alternative per-sample clustering available in [scripts/collapser_otu95_per_sample.sh](scripts/collapser_otu95_per_sample.sh)

**Impact:** May miss rare sample-specific taxa but enables cross-sample analyses

### CLR Transformation for Networks
**Decision:** Apply centered log-ratio (CLR) before correlation analyses  
**Rationale:**
- Metabarcoding counts are compositional (relative abundances sum to constant)
- Raw correlations on compositional data are mathematically spurious
- CLR transformation opens the simplex to real space, enabling valid Pearson/Spearman correlations
- Established best practice in microbiome co-occurrence studies

**Documentation:** Clearly stated in thesis that networks are correlational, not causal

### Conservative Network Threshold |r|≥0.60
**Decision:** Use stringent correlation cutoff for primary networks  
**Rationale:**
- Small sample size (n=4 including unclassified) requires conservative thresholds
- Reduces false positive edges from chance correlations
- Sensitivity analyses at |r|≥0.50, 0.55, 0.65 tested

**Caveat:** Exploratory only; no claim of statistical significance or biological causality

---

## Tools & Versions

### Basecalling & Demultiplexing
- **Dorado** 1.3.0
  - Model: `dna_r10.4.1_e8.2_400bps_hac@v4.3.0` (HAC accuracy)
  - Hardware: CUDA 11+ GPU (tested: NVIDIA RTX 3060 Ti)
  - Location: [dordowna/bin/dorado](dordowna/bin/dorado)

### Quality Control
- **Porechop** - Oxford Nanopore adapter trimming
- **Chopper** - ONT read filtering (quality, length)
- **seqkit** 2.12.0 - FASTQ statistics and manipulation

### Taxonomic Classification
- **EMU** - Expectation-Maximization classifier
  - Type: `map-ont` (optimized for long reads)
  - Flags: `--keep-counts`, `--keep-files`
  - Database tool included for custom reference builds

### Workflow Management
- **Nextflow** ≥23.0 (DSL2 pipelines)
  - Main workflow: [sixtran/main.nf](sixtran/main.nf)
  - Configuration: [sixtran/nextflow.config](sixtran/nextflow.config)
- **Java** 17 (Nextflow dependency)

### Sequence Analysis
- **Vsearch** 2.12.0 - OTU clustering, dereplication
- **Minimap2** - Reference alignment (alternate workflows)
- **Samtools** - BAM manipulation

### Python Environment
**Core packages:**
- Python 3.10-3.12
- pandas, numpy (data manipulation)
- matplotlib, seaborn (visualization)
- scikit-bio (diversity metrics, ordination)
- scipy (statistical functions, correlations)

**Utilities:**
- csvkit - TSV/CSV manipulation
- parallel - GNU parallel for batch processing
- pigz - Parallel gzip compression
- curl, wget - File downloads

**Environment files:**
- [sixtran/funcall.yml](sixtran/funcall.yml) - Full Nextflow stack (recommended)
- [env/funcall.yml](env/funcall.yml) - Lightweight prototype environment

### Custom Scripts
**Core workflow:** [scripts/](scripts/)
- [collapser_build.py](scripts/collapser_build.py) - Build OTU input tables from EMU results
- [collapser_otu95.sh](scripts/collapser_otu95.sh) - Bash wrapper for comparative OTU clustering
- [collapser_cluster_memberships.py](scripts/collapser_cluster_memberships.py) - Generate taxid→cluster mappings

**Nextflow helpers:** [sixtran/bin/](sixtran/bin/)
- [format_emu_table.py](sixtran/bin/format_emu_table.py) - Normalize EMU output format
- [merge_emu_tables.py](sixtran/bin/merge_emu_tables.py) - Combine per-sample EMU tables

**Component analyses:** [parrot/components/](parrot/components/), [pukeko/components/](pukeko/components/)
- Diversity metrics, ordination, visualization scripts

---

## Troubleshooting

### CUDA/GPU Errors
**Symptom:** `CUDA not found` or `No GPU detected` during Dorado basecalling  
**Solutions:**
- Check GPU visibility: `nvidia-smi`
- Verify CUDA drivers: `nvcc --version` (require CUDA 11+)
- CPU-only fallback: Remove GPU flags (20× slower)

### Memory Issues
**Symptom:** `Killed` or `Out of memory` during EMU or clustering  
**Solutions:**
- Verify RAM: `free -h` (require 32GB minimum)
- Reduce parallel processes in Nextflow config
- Process barcodes sequentially instead of parallel

### Conda Environment Conflicts
**Symptom:** Package version conflicts during environment creation  
**Solutions:**
- Use `mamba` instead of `conda` (faster dependency resolution)
- Update conda/mamba: `conda update -n base conda`
- Create environment from scratch: `mamba env remove -n funcall && mamba env create -f sixtran/funcall.yml`

### Broken Symlinks
**Symptom:** `No such file or directory` for `house/input`, `mapping/refdbs`, or `dordowna`  
**Solutions:**
- Verify symlink targets exist: `ls -l house/input mapping/refdbs dordowna`
- Check `data/` location: `ls -l ../data/refs/funcall/mapping/refdbs`
- Recreate symlinks if `data/` moved:
  ```bash
  ln -sfn /new/path/data/raw/funcall/house/input house/input
  ln -sfn /new/path/data/refs/funcall/mapping/refdbs mapping/refdbs
  ln -sfn /new/path/data/tools/dordowna dordowna
  ```

### Nextflow Execution Errors
**Symptom:** `Process died`, `Unable to find Java`, or config parse errors  
**Solutions:**
- Verify Java 17: `java -version`
- Check Nextflow version: `nextflow -version` (require ≥23.0)
- Validate config syntax: `nextflow config sixtran/nextflow.config`
- Clean work directory: `rm -rf work/` and rerun

### Empty OTU Tables
**Symptom:** OTU table has zero rows or all-zero counts  
**Solutions:**
- Check read thresholds: Lower from ≥10 to ≥5 or ≥1 for trace mode
- Verify EMU database integrity: `ls mapping/refdbs/mergemimt/emu_db/`
- Confirm input FASTQs not empty: `seqkit stats oven/slashing/morris/filtered/*.fastq`
- Review collapser table: `head parrot/rout/collapser/collapser_abundance.tsv`

### Low Taxonomic Coverage
**Symptom:** Very few taxa detected by EMU  
**Solutions:**
- Try alternative database: UNITE ITS in [mapping/refdbs/unit/emu_db](mapping/refdbs/unit/emu_db)
- Lower QC thresholds: Reduce MINQ from 15 to 12 or MINLEN from 1000 to 800
- Check amplicon target match: Verify primers amplified ITS/18S region

---

## Comparison to Sixteen Pipeline

The funcall (fungal) and sixteen (bacterial) pipelines share a common analytical framework but differ in amplicon biology and contamination profiles.

### Shared Components
- **Collapser scripts:** Both use [scripts/collapser_build.py](scripts/collapser_build.py) and [scripts/collapser_otu95.sh](scripts/collapser_otu95.sh)
- **OTU clustering:** Vsearch with comparative mode for cross-sample analyses
- **Network inference:** CLR transformation + Pearson/Spearman correlations
- **Diversity metrics:** Alpha (Shannon, Chao1) and beta (Bray-Curtis) in component scripts
- **Thesis integration:** Both contribute complementary microbial community data to [writeup/](../writeup/)

### Key Differences

| Feature | Funcall (Fungi) | Sixteen (Bacteria) |
|---------|-----------------|-------------------|
| **Amplicon** | ITS/18S rRNA | 16S rRNA |
| **Read length** | 5.0-5.6 kb (median ~5.2 kb) | ~1.5 kb |
| **Basecaller** | Dorado HAC (GPU) | Dorado SUP/HAC |
| **Database** | MIMt 18S+ITS (61K) or UNITE (100K) | SILVA/RDP/Greengenes |
| **Clustering** | 95% (genus-level) | 95% or 97% (near species-level) |
| **Contamination** | 32.9% non-fungal (plants, animals, oomycetes) | Lower non-bacterial signal |
| **QC retention** | ~42% (Q≥15, 1-7 kb) | Higher retention (shorter amplicons) |
| **OTU count** | 411 (all-taxa) or 83 (fungi-only) | 192 (threepoint filter) |
| **Target list** | 51 fungal genera (6 functional categories) | 14 bacterial genera (dioxin degraders) |
| **Detection rate** | 15/51 genera (29.4%) | 18 OTUs matching target genera |

### Integration Strategy
- **Bacterial (sixteen):** Establishes cohort structure (fuzzsample vs drysample) and core POP-degrader presence
- **Fungal (funcall):** Characterizes symbiotic guilds (ECM, AMF), decomposer diversity, and white-rot potential
- **Combined networks:** Exploratory bacteria-fungi co-occurrence (future direction, not in current thesis scope)

**See also:** [sixteen/README.md](../sixteen/README.md) for bacterial 16S workflow details.

---

## References

### Tools
- **Dorado:** Oxford Nanopore Technologies basecaller. https://github.com/nanoporetech/dorado
- **EMU:** Expectation-Maximization classifier for multi-taxonomy mapping. Krause, Lesker, et al. (2020). https://gitlab.com/treangenlab/emu
- **Vsearch:** Versatile sequence analysis toolkit. Rognes et al. (2016). PeerJ 4:e2584. https://github.com/torognes/vsearch
- **Nextflow:** Workflow management system. Di Tommaso et al. (2017). Nature Biotechnology 35:316-319. https://www.nextflow.io/
- **Chopper:** Oxford Nanopore read filtering. https://github.com/wdecoster/chopper
- **Porechop:** ONT adapter trimmer. https://github.com/rrwick/Porechop

### Databases
- **MIMt:** Microbial Metagenomic Taxonomic databases (18S + ITS). https://github.com/aemann01/MIMT
- **UNITE:** Fungal ITS reference database. Nilsson et al. (2019). Nucleic Acids Research 47:D259-D264. https://unite.ut.ee/
- **NCBI Taxonomy:** Taxonomic classification framework. https://www.ncbi.nlm.nih.gov/taxonomy

### Methods
- **CLR transformation:** Aitchison, J. (1986). The Statistical Analysis of Compositional Data. Chapman & Hall.
- **Compositional data analysis:** Gloor et al. (2017). Frontiers in Microbiology 8:2224.
- **Co-occurrence networks:** Berry & Widder (2014). Trends in Microbiology 22:245-247.

### Study Context
- **anonymized contaminated site:** Historical TCDD contamination from chlorophenol production, New Zealand
- **POP degradation:** Persistent organic pollutant bioremediation (lignin-degrading fungi, bacterial degraders)
- **Metabarcoding best practices:** Nilsson et al. (2019). Molecular Ecology Resources 19:14-20.

---

## Additional Documentation

For detailed operational notes, see component READMEs:
- [house/README.md](house/README.md) - Basecalling and demux commands with dated admin logs
- [house/AGENTS.md](house/AGENTS.md) - Basecalling troubleshooting and agent-assisted workflows
- [oven/slashing/README.md](oven/slashing/README.md) - QC filter development history (Morris rounds 1-2)
- [mapping/README.md](mapping/README.md) - EMU database construction and mapping workflow
- [oven/snipe/README.md](oven/snipe/README.md) - Target list curation and pull-down methodology
- [parrot/README.md](parrot/README.md) - All-taxa OTU workflow and component analyses
- [pukeko/README.md](pukeko/README.md) - Fungi-only refinement rationale and filtering scripts
- [oven/shroom/README.md](oven/shroom/README.md) - Non-OTU fungal analysis (species-level, no clustering)
- [sixtran/primer.md](sixtran/primer.md) - Pipeline conceptual overview and threshold decisions
- [sixtran/where-we-are.md](sixtran/where-we-are.md) - Current pipeline state and active development

For thesis context and results integration:
- [writeup/RESULTS_PLAN_FUNCALL.md](../writeup/RESULTS_PLAN_FUNCALL.md) - Planned fungal results sections
- [writeup/RESULTS_PLAN.md](../writeup/RESULTS_PLAN.md) - Overall thesis structure (16S + funcall + Roadtrip)
- [writeup/RESULTS_SECTION_DRAFT.md](../writeup/RESULTS_SECTION_DRAFT.md) - Draft results text

For workspace organization:
- [../README.md](../README.md) - Monorepo structure and data management policy
- [../WORKSPACE_NOTES.md](../WORKSPACE_NOTES.md) - Git policy and symlink conventions
