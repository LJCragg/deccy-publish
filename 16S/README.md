# sixteen

**16S rRNA amplicon sequencing analysis pipeline for Oxford Nanopore Technologies (ONT) data**

Performs taxonomic classification and OTU-level clustering of bacterial communities with specialized focus on dioxin/POP-degrader associated taxa.

**Last updated:** 2026-02-20  
**Current dataset:** 6 soil samples (barcodes 13-18) representing two cohorts (fuzzsample: 13-15, drysample: 16-18)

---

## Project Context: The anonymized site Dioxin Degradation Study

This 16S bacterial amplicon pipeline is one component of a **three-pronged metagenomic investigation** of microbial communities in dioxin-contaminated soils from the **anonymized contaminated site** in anonymized New Zealand site, New Zealand.

### Site Background

The anonymized contaminated site was contaminated with **2,3,7,8-Tetrachlorodibenzo-p-dioxin (TCDD)** and related chlorophenols as byproducts of herbicide production by anonymized contaminated site between **1962 and 1987**. This persistent organic pollutant (POP) contamination created selective pressure for specialized microbial communities capable of degrading chlorinated aromatic compounds.

### Three Complementary Approaches

This study employs three independent sequencing strategies to comprehensively characterize the microbial degradation potential:

#### 1. **16S rRNA Bacterial Amplicon Sequencing** (THIS PIPELINE)
- **Purpose:** Community-level taxonomic profiling of bacteria
- **Technology:** Oxford Nanopore MinION (R10.4.1, SUP basecalling)
- **Samples:** 6 barcodes (13-18) representing two soil cohorts
  - **fuzzsample:** barcodes 13, 14, 15 (n=3 biological replicates)
  - **drysample:** barcodes 16, 17, 18 (n=3 biological replicates)
- **Key Outcomes:**
  - 437 bacterial taxids detected (436 species-level)
  - 192 OTU95 features after reproducibility filtering
  - 18 degrader-associated OTU clusters from 14 target genera
  - CLR-based co-occurrence networks
- **Documentation:** This README, [primer.md](primer.md)

#### 2. **Fungal ITS/18S Amplicon Sequencing** ([../funcall/](../funcall/))
- **Purpose:** Fungal community structure and POP-degrader guild identification
- **Samples:** 3 barcodes (19-21) representing fungal amplicons
- **Key Outcomes:**
  - 1,306 eukaryotic taxa detected (876 fungal entries, 67% of signal)
  - 83 fungal OTUs at 95% identity
  - Detection of 15/51 target genera (mycorrhizal, white-rot fungi)
- **Documentation:** [../funcall/README.md](../funcall/README.md)

#### 3. **Targeted Functional Gene Sequencing** ([../roadtrip/](../roadtrip/))
- **Purpose:** Direct detection and diversity characterization of dioxin degradation genes
- **Technology:** PCR amplification + ONT sequencing + Medaka consensus calling
- **Samples:** 2 soil DNA extracts across 4 barcodes (01-04)
- **Target Genes:** clcA, catB2, BpHc, ntDAa (degradation pathway enzymes)
- **Key Outcomes:**
  - clcA: 11,923 reads → 7 consensus sequences (*Pseudomonas*, *Diaphorobacter*)
  - catB2: 2,521 reads → 6 consensus sequences (*Sphingobium*)
  - Multiple sequence alignment: 576 bp conserved core identified
- **Documentation:** [../roadtrip/README.md](../roadtrip/README.md)

### Integration Strategy: Linking Community to Function

The three approaches provide complementary evidence:

```
┌─────────────────────────────────────────────────────────────────┐
│  16S Amplicon (Community Structure)                             │
│  ↓ Identifies potential degrader taxa at genus level            │
│  ↓ Example: Sphingomonas, Pseudomonas, Desulfitobacterium       │
└─────────────────────────┬───────────────────────────────────────┘
                          │
                          ↓ TAXA-TO-FUNCTION LINKAGE
                          │
┌─────────────────────────┴───────────────────────────────────────┐
│  Targeted Gene Sequencing (Functional Capacity)                 │
│  ↓ Confirms presence of degradation genes                       │
│  ↓ Example: clcA from Pseudomonas, catB2 from Sphingobium       │
└─────────────────────────┬───────────────────────────────────────┘
                          │
                          ↓ GENE DIVERSITY
                          │
┌─────────────────────────┴───────────────────────────────────────┐
│  Multiple Sequence Alignment (Gene Variants)                    │
│  ↓ Characterizes functional gene diversity                      │
│  ↓ Example: 13 consensus sequences, 576 bp conserved core       │
└─────────────────────────────────────────────────────────────────┘
```

**Key Research Questions:**
1. Does the bacterial community contain **known dioxin-degrading taxa**? (16S)
2. Are **functional degradation genes** actually present? (Targeted sequencing)
3. How **diverse** are these genes and their carriers? (Consensus + alignment)
4. Do taxa and genes **co-occur** predictably? (Network analysis)

### Monorepo context

- Repo root: `/home/uca/chover`
- Large inputs and reference DBs live under `data/` and are symlinked into `rin/hq` and `refdbs` (gitignored).
- Workspace policy lives in `README.md` at repo root.

### Pipeline Background

This 16S pipeline implements a full amplicon workflow for ONT reads, built around the **Emu classifier** and a custom **"collapser" OTU layer**. Raw barcoded FASTQs from two soil cohorts (fuzzsample and drysample) are classified against the curated **MIMt 16S database** (47k species-level references), then clustered into OTU-like groups at **95%** (genus-level) and **97%** (species-level) identity. 

Post-hoc filters (**threepoint**, **core**, **associated**) focus the analysis on reproducible, abundant taxa and a predefined shortlist of dioxin/POP-degrader genera. The downstream **R layer** performs compositional (CLR) analysis and network inference on these filtered OTUs, producing publication-grade figures and tables.

**Critical Distinction:** This README separates **EMU pre-OTU metrics** (direct taxonomic classification) from **OTU post-clustering metrics** (operational units at 95%/97% similarity). Diversity indices and richness estimates are reported for both levels with clear labeling.

---

## Table of Contents

- [Study Design & Sample Information](#study-design--sample-information)
- [Key Results Summary](#key-results-summary)
- [Quick Start](#quick-start)
- [Installation](#installation)
- [Pipeline Overview](#pipeline-overview)
- [Read Retention & QC Summary](#read-retention--qc-summary)
- [Directory Structure Explained](#directory-structure-explained)
- [Key Workflows](#key-workflows)
- [Output Files Explained](#output-files-explained)
- [Targeted Gene Integration](#targeted-gene-integration)
- [Methodological Notes & Critical Assessment](#methodological-notes--critical-assessment)
- [Figures & Tables Reference](#figures--tables-reference)
- [Usage Examples](#usage-examples)
- [Troubleshooting](#troubleshooting)

---

## Study Design & Sample Information

### Sample Origin and Cohort Structure

**Source:** anonymized site/anonymized contaminated site TCDD-contaminated soil samples, New Zealand

**Cohort Definitions:**
- **fuzzsample (BC13-15):** n=3 biological replicates from soil sample A
- **drysample (BC16-18):** n=3 biological replicates from soil sample B

**Important context:** Cohort labels (fuzzsample/drysample) are barcode-defined based on sample groupings. No separate sample metadata file documents site-specific covariates (e.g., contamination levels, depth, extraction batch). This is a **limitation** for inferential analysis—treat cohort comparisons as exploratory.

### Biological Replication Strategy

**Design:** n=3 technical/biological replicates per cohort
- **Rationale:** Minimum replication for variance estimation and reproducibility assessment
- **Limitation:** Small n restricts statistical power—emphasize effect sizes over p-values
- **Reproducibility filter:** "Threepoint" filter requires OTU presence in all 3 replicates within ≥1 cohort

### Sample Processing

1. **DNA extraction:** Soil DNA extracted from anonymized site samples
2. **Library prep:** 16S V1-V9 full-length amplicon (PCR primers targeting ~1500 bp)
3. **Barcoding:** Native barcoding kit (ONT SQK-NBD114-24)
4. **Sequencing:** MinION R10.4.1 flowcell, SUP basecalling (Dorado)
5. **Subsampling:** 120,000 reads per barcode (normalized for depth)

### Cross-Project Sample Mapping

**Question:** Do 16S barcodes (BC13-18) correspond to targeted gene barcodes (BC01-04)?

**Status:** **DIFFERENT SAMPLES** — Confirmed as separate sampling events
- **16S samples (BC13-18):** Six soil replicates across two cohorts (fuzzsample, drysample)
- **Targeted gene samples (BC01-04):** Two soil extracts (SL-1A, SL-2A) with technical replicates

**Integration strategy:** Treat as **two complementary approaches** to the same contaminated site
- Compare results at genus/gene level (e.g., "Both methods detect Sphingomonas")
- DO NOT link at sample level (e.g., avoid "BC13 has Sphingomonas with catB2")
- Valid integration: "16S detected degrader taxa; targeted sequencing confirmed functional genes from same genera"

---

## Key Results Summary

### Overview Statistics

| Metric | Value | Description |
|--------|-------|-------------|
| **Raw reads** | 120,000/barcode | Subsampled for normalization |
| **Filtered reads** | ~47,700/barcode | Post-QC (top 40% mean-Q + 500–2000 bp + 15 bp end-crop) |
| **EMU taxids detected** | 437 unique | Species-level NCBI taxonomy |
| **EMU genera detected** | ~250 | Genus-level richness |
| **OTU95 clusters** | 258 | Genus-level (95% identity) |
| **OTU97 clusters** | 353 | Species-level (97% identity) |
| **Threepoint OTUs** | 192 (OTU95) | Reproducible across replicates |
| **Core OTUs** | 55 (OTU95) | Prevalent + abundant (≥0.5%) |
| **Degrader OTUs** | 18 (OTU95) | Matched to 14 target genera |

### Community Structure (Phylum Level)

**Dominant phyla:**
1. **Bacillota:** 72.9 ± 6.8% (fuzz), 64.6 ± 2.8% (dry)
   - Higher in fuzzsample with greater variance
   - Consistent with stress-tolerant, spore-forming lineages
2. **Pseudomonadota:** 24.1 ± 1.3% (fuzz), 23.5 ± 0.2% (dry)
   - Stable across cohorts
   - Metabolically versatile heterotrophs
3. **Bacteroidota:** 1.1 ± 0.1% (fuzz), 8.4 ± 1.1% (dry)
   - **7.6-fold depletion** in fuzzsample
   - Associated with complex carbon degradation

**Top 3 phyla account for >99% of community in all samples.**

### Diversity Metrics (FACT-CHECKED)

**⚠️ CRITICAL DISTINCTION:** Diversity metrics differ between EMU (pre-OTU) and OTU (post-clustering) levels.

#### EMU Genus-Level Diversity (Pre-OTU)
Based on direct taxonomic classification of 437 taxids:
- **fuzzsample Shannon:** 3.58 ± 0.26 (range: 3.28–3.80)
- **drysample Shannon:** 4.55 ± 0.11 (range: 4.41–4.64)
- **Interpretation:** drysample has higher genus-level diversity in raw classifications

#### OTU97 Diversity (Post-Clustering)
Based on 97% sequence similarity clustering:
- **fuzzsample Shannon:** 2.56 ± 0.06 (range: 2.52–2.64)
- **drysample Shannon:** 2.79 ± 0.17 (range: 2.61–2.96)
- **Interpretation:** drysample retains higher diversity after clustering, but values are lower due to sequence collapsing

**Why the difference?** EMU reports taxonomic diversity from database matches (many-to-many mapping). OTU clustering collapses similar sequences, reducing apparent diversity. **Both metrics are valid but measure different aspects** of community structure.

**Recommendation:** Report both levels in results, clearly labeled. Use OTU metrics for ecological inference (operational units), EMU metrics for taxonomic richness.

### Degrader-Associated Taxa

**Detection method:** Genus-level matching to predefined whitelist ([refdbs/dioxin_pop_degrader_genera.txt](refdbs/dioxin_pop_degrader_genera.txt))

**Results (OTU95 threepoint-filtered data):**
- **18 OTU clusters** matched 14 target degrader genera
- **Detection rate:** 100% (present in all 6 samples)
- **Dominant genera:**
  1. **Sphingomonas** (8 OTUs, mean 1,500 reads/sample)
  2. **Novosphingobium** (4 OTUs)
  3. **Desulfitobacterium** (2 OTUs, reductive dechlorination specialists)
  4. **Acetobacterium** (1 OTU, acetogenic metabolism)

**Integration with targeted genes:**
- 16S detected *Sphingomonas* → Playground confirmed **catB2** gene from *Sphingobium*
- 16S detected *Pseudomonas*/*Diaphorobacter* → Playground confirmed **clcA** gene
- **Taxa-to-function linkage supported** (but not absolute proof without cultivation)

### Co-Occurrence Networks

**Analysis:** CLR-transformed degrader-associated OTUs (18 clusters, 6 samples)
- **Method:** Pearson correlation on CLR values, Louvain community detection
- **Threshold:** r ≥ 0.6 for main network (sensitivity tested: 0.5, 0.7)
- **Result:** [Exploratory network graph](R/aimed/figures/fig_associated_louvain_r0.60.png)

**⚠️ Limitation:** Small sample size (n=6) means network is **hypothesis-generating only**. Correlations may be spurious—do not interpret as ecological interactions without independent validation.

---

## Quick Start

```bash
# 1. Activate environment
conda activate sixteen

# 2. Run QC pipeline  
cd scripts/ && bash cleaner.sh

# 3. Run Emu classification
cd ../workflows/emu-nf/
nextflow run main.nf --reads ../../rin/filtered/*.fastq --db ../../refdbs/mimt/emu_db

# 4. Build collapser table
cd ../../scripts/
python collapser_build.py --emu-table ../rout/emu/emu_abundance.tsv --min-reads 10 --out ../rout/collapser/collapser_abundance.tsv

# 5. OTU clustering at 95%
bash collapser_cluster_0.95.sh

# 6. Filter to reproducible OTUs
python filter_threepoint_core.py ../cannon/collapsed95/collapser_abundance_0.95.tsv ../oven/slashing/threepoint/

# 7. Filter to core microbiome
python filter_core_otus.py ../oven/slashing/threepoint/collapser_abundance_0.95_threepoint.tsv ../oven/slashing/core_p2_mean0.5/

# 8. Extract degrader-associated OTUs
python filter_associated_otus.py ../oven/slashing/threepoint/collapser_abundance_0.95_threepoint.tsv ../refdbs/dioxin_pop_degrader_genera.txt ../oven/associated/

# 9. Run compositional analysis (R)
cd ../R/scripts/
Rscript shep1_associated_clr.R
Rscript shep2_associated_clr_long_taxa.R
Rscript shep3_threepoint_sample_cor.R
Rscript shep4_threepoint_network.R
```

---

## Installation

### Environment Setup

```bash
# Create from template (use sixtran/sixteen.yml if env/sixteen.yml missing)
conda env create -f env/sixteen.yml  
conda activate sixteen

# Verify dependencies
emu --version          # Taxonomic classifier
vsearch --version      # OTU clustering
seqkit version         # FASTA manipulation
porechop --version     # Adapter trimming
chopper --version      # Quality filtering
nextflow -version      # Pipeline runner
```

### Reference Database

```bash
cd refdbs/mimt/
bash scripts/build_mimt_emu.sh  # Builds Emu database from MIMt 16S reference
```

**Database:** MIMt 16S (47k curated full-length 16S sequences, species-level resolution)

---

## Pipeline Overview

### Full Pipeline with Quantitative Milestones

```
┌─────────────────────────────────────────────────────────────┐
│  INPUT: RAW READS (subsampled for normalization)            │
│  Location: rin/hq/*.fastq                                   │
│  Reads: 120,000 per barcode × 6 barcodes = 720,000 total    │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ↓
┌─────────────────────────────────────────────────────────────┐
│  STAGE 1: QUALITY CONTROL (scripts/run_porechop_chopper.sh)│
│  Tools: Porechop (adapters) + Chopper (Q-filter)           │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ↓ Retention: ~39.7% (top 40% mean-Q by design)
                      │
                      ↓ rin/filtered/*.fastq
                      ↓ ~47,700 reads/barcode × 6 = ~286,000 total
                      │
┌─────────────────────┴───────────────────────────────────────┐
│  STAGE 2A: TAXONOMIC CLASSIFICATION (workflows/emu-nf/)     │
│  Tool: Emu (MIMt 16S database, 47k references)             │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ↓ rout/emu/emu_abundance.tsv
                      ↓ 437 unique taxids (436 species-level)
                      ↓ ~250 genera across all samples
                      │
┌─────────────────────┴───────────────────────────────────────┐
│  STAGE 2B: COLLAPSER TABLE (scripts/collapser_build.py)    │
│  Function: Normalize, filter low-abundance taxa (≥10 reads) │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ↓ rout/collapser/collapser_abundance.tsv
                      ↓ Analysis-grade table (min 10 reads)
                      │
┌─────────────────────┴───────────────────────────────────────┐
│  STAGE 2C: OTU CLUSTERING (scripts/collapser_cluster_*.sh) │
│  Tool: vsearch --cluster_fast                              │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ├→ 95% identity (genus-level)
                      │  cannon/collapsed95/*.tsv
                      │  258 OTU clusters
                      │  42% singletons
                      │
                      └→ 97% identity (species-level)
                         cannon/collapsed97/*.tsv
                         353 OTU clusters
                         69% singletons
                      │
┌─────────────────────┴───────────────────────────────────────┐
│  STAGE 2D: BIOLOGICAL FILTERING (scripts/filter_*.py)      │
│  Purpose: Remove noise, focus on reproducible/abundant taxa │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      ├→ THREEPOINT FILTER
                      │  Logic: Present in all 3 reps of ≥1 cohort
                      │  oven/slashing/threepoint/
                      │  258 → 192 OTU95 clusters (74% retention)
                      │  Purpose: Reproducibility across replicates
                      │
                      ├→ CORE FILTER (applied to threepoint output)
                      │  Logic: ≥2/3 reps + ≥0.5% mean abundance
                      │  oven/slashing/core_p2_mean0.5/
                      │  192 → 55 OTU95 clusters (29% of threepoint)
                      │  Purpose: Prevalent AND numerically dominant
                      │
                      └→ ASSOCIATED FILTER (applied to threepoint output)
                         Logic: Genus matches degrader whitelist
                         oven/associated/
                         192 → 18 OTU95 clusters (9% of threepoint)
                         14 genera matched (Sphingomonas, Desulfitobacterium, etc.)
                         Purpose: Dioxin/POP-degrader candidates
                      │
┌─────────────────────┴───────────────────────────────────────┐
│  STAGE 3: COMPOSITIONAL ANALYSIS (R/scripts/shep*.R)       │
│  Method: CLR transformation + correlation networks          │
└─────────────────────────────────────────────────────────────┘
                      │
                      ↓ CLR-transformed abundances
                      ↓ R/aimed/long_associated_clr.tsv
                      │
                      ↓ Correlation matrices (Pearson on CLR values)
                      ↓ R/outputs/pearson/correlation_matrix.tsv
                      │
                      ↓ Network graphs (Louvain communities)
                      ↓ R/aimed/figures/fig_associated_louvain_r0.60.png
```

### Which Clustering Resolution Should I Use?

| Resolution | Identity | OTU Count | Singleton % | Best For |
|------------|----------|-----------|-------------|----------|
| **OTU95** | 95% | 258 → 192 (threepoint) | 42% | Genus-level community trends, broad taxonomic patterns |
| **OTU97** | 97% | 353 → 245 (threepoint) | 69% | **Recommended for degrader analysis** — species-level resolution for target taxa |

**Recommended Strategy:**
- **EMU abundance table:** Raw read findings, initial taxonomic profiling (437 taxids, pre-clustering)
- **OTU95:** Genus-level community trends, phylum composition, broad ecological patterns
- **OTU97:** Species-level analysis, especially for **degrader-associated taxa** after taxonomy filtering (a priori target list)

**Rationale for OTU97 in degrader analysis:** When focusing on a predefined target list (14 degrader genera), species-level resolution provides better discrimination between closely related strains with potentially different functional capacities. Post-taxonomy filtering reduces noise from singleton OTUs.

### Which Filter Should I Use?

| Filter | Input → Output | Purpose | Use When |
|--------|----------------|---------|----------|
| **Threepoint** | 258 → 192 OTUs | Reproducibility | You want OTUs present in all replicates of at least one cohort (removes transient/contaminant taxa) |
| **Core** | 192 → 55 OTUs | Dominance | You want only the most prevalent AND abundant OTUs (≥2/3 reps, ≥0.5% mean abundance within cohort) |
| **Associated** | 192 → 18 OTUs | Functional targeting | You want degrader-associated genera only (requires predefined genus whitelist) |

**Recommendation:** Start with **threepoint** for exploratory analysis. Apply **associated** filter for degrader-specific questions. Use **core** to focus on dominant community drivers.

---

## Read Retention & QC Summary

### Pipeline Efficiency by Stage

| Stage | Description | Input | Output | Retention | Quality Δ | Purpose |
|-------|-------------|-------|--------|-----------|-----------|---------|
| **0. Subsampling** | Normalize read depth | Variable | 120,000/BC | 100% | N/A | Fair comparison across samples |
| **1. Porechop** | Adapter trim/split | 120,000 | (not summarised here) | N/A | N/A | Remove/split adapter-bearing reads prior to Q filtering |
| **2. Chopper** | Quality + length filter | 120,000 | ~47,700 | ~39.7% | N/A | Top 40% mean-Q reads (per-sample cutoff), 500-2000 bp after fixed 15 bp crop |
| **3. EMU mapping** | Taxonomic classification | ~47,700 | ~47,500 | ~99% | N/A | Assign reads to 437 taxids |
| **4. OTU clustering** | Sequence similarity | 437 taxids | 258 OTUs | N/A | N/A | Group at 95% identity (genus) |
| **5A. Threepoint** | Reproducibility filter | 258 OTUs | 192 OTUs | 74% | N/A | Present in all 3 reps/cohort |
| **5B. Core** | Prevalence + abundance | 192 OTUs | 55 OTUs | 29% | N/A | ≥2/3 reps + ≥0.5% mean |
| **5C. Associated** | Degrader whitelist | 192 OTUs | 18 OTUs | 9% | N/A | Match target genera |

### Key Insights

**Dominant filter:** the top-40% mean-Q cutoff
- By construction, ~60% of reads fall below the per-sample Q cutoff.
- Length bounds (500–2000 bp after cropping) remove a smaller additional fraction.

**High EMU mapping rate:** 95% of filtered reads successfully classified
- Indicates good amplicon quality and database coverage
- MIMt 16S database (47k references) provides comprehensive species-level matching

**OTU clustering efficiency:** 437 taxids → 258 OTU95 clusters
- **1.69 taxids per cluster** on average
- 42% singletons (1 taxid = 1 OTU) indicate many unique species
- 58% multi-member clusters indicate genus-level collapsing working as expected

**Threepoint filter is permissive:** 74% retention (192/258)
- Most OTUs reproducible across replicates
- 26% loss likely represents transient/low-abundance taxa

**Core filter is stringent:** 29% retention (55/192)
- Focuses on numerically dominant taxa (≥0.5% mean abundance)
- 71% loss expected—removes reproducible but rare OTUs

---

## Top-Level Directory Cheat Sheet

| Directory | Purpose | Typical Contents |
|-----------|---------|------------------|
| `awake/` | Lightweight CLR + network workflow (Python-based alternative to full R pipeline) | Scripts to make CLR matrices and correlation edges directly from OTU tables |
| `cache/` | Nextflow / tooling cache | Transient files; can usually be regenerated or cleaned |
| `cannon/` | OTU clustering sandbox | Comparative and per-sample OTU tables at 95% and 97% identity, membership maps, centroids, clustering comparison notes |
| `components/` | **Taxonomic diversity analysis** | Baseline diversity metrics (alpha/beta diversity, richness, statistical tests) for EMU and filtered datasets with publication-ready outputs |
| `env/` | Environment definitions | `sixteen.yml` Conda env specification used by this project |
| `Luca/` | Project-specific extra analyses / notebooks (if populated) | Ad-hoc exploration, not part of core pipeline |
| `oven/` | Biologically filtered OTU outputs | Threepoint/core/associated OTU tables at 95% and 97%, plus experimental filters (including brush/refined) |
| `R/` | Stage 3 R analysis | R project, scripts, CLR matrices, correlation matrices, and figures |
| `reading/` | Background reading and notes | Literature summaries, threshold rationales, citations |
| `refdbs/` | Reference databases | MIMt Emu DB, NCBI taxonomy, degrader genus whitelist |
| `rin/` | Raw and QC’d reads | `rin/hq/` (immutable raw FASTQs), `rin/filtered/` (porechop+chopper outputs) |
| `rout/` | Reference (official) pre-OTU outputs | Emu abundance tables and collapser analysis/trace tables used as canonical inputs to clustering |
| `scripts/` | Entry-point scripts and utilities | QC scripts, collapser build, clustering, and filtering helpers |
| `work/` | Nextflow and other workflow scratch space | Per-run work directories; safe to clean with `nextflow clean` when jobs finish |
| `workflows/` | Nextflow workflows | Emu DSL2 pipeline and any additional NF-based components |

The usual flow is: `rin/` (reads) → `workflows/` (pipelines) → `rout/` (reference outputs) → `cannon/` (OTU experiments) → `oven/` (filtered OTUs) → `R/` / `awake/` (statistics & visualisation).

---

## Directory Structure Explained

### Input Data (rin/)

**rin/hq/**  
- **Type:** Raw sequencer output  
- **OTU status:** ❌ No  
- **Contents:** Demultiplexed ONT FASTQs (barcodes 13-18)  
- **Biology:** Full-length 16S rRNA amplicons (~500-2000bp) from soil bacterial communities  
- **Status:** **IMMUTABLE** — Never modify

**rin/filtered/**  
- **Type:** QC-processed reads  
- **OTU status:** ❌ No  
- **Contents:** Adapter-trimmed, quality-filtered FASTQs  
- **Biology:** Clean reads for taxonomic classification (top 40% Q-scores, length 500-2000bp)  
- **Generated by:** `scripts/cleaner.sh`

---

### Reference Outputs (rout/)

**Purpose:** Official/reference outputs from Emu and collapser table build

**rout/emu/**  
- **Type:** Species-level taxonomic classification  
- **OTU status:** ❌ No (pre-OTU)  
- **Contents:** Per-sample Emu abundance tables + combined `emu_abundance.tsv`  
- **Biology:** Direct taxonomic assignments from Emu classifier (436 unique taxa detected across 6 samples)  
- **Columns:** `sample_id`, `tax_id`, `estimated counts`, taxonomy ranks (kingdom→species)  
- **Project role:** Starting point for all downstream analysis

**rout/collapser/**  
- **Type:** Normalized taxonomy (analysis-grade)  
- **OTU status:** ❌ No (pre-OTU)  
- **Contents:** `collapser_abundance.tsv` (min-reads ≥10)  
- **Biology:** Filtered Emu table with noise removal (low-abundance taxa removed)  
- **Project role:** Input to OTU clustering (vsearch)

**rout/collapser_trace/**  
- **Type:** Normalized taxonomy (trace-grade)  
- **OTU status:** ❌ No (pre-OTU)  
- **Contents:** `collapser_abundance.tsv` (min-reads ≥1, includes singletons)  
- **Biology:** Sensitive version retaining rare taxa for diversity analysis  
- **Project role:** Parallel sensitivity analysis track

---

### Experimental Sandbox (cannon/)

**Purpose:** OTU clustering experiments and testing ground

**cannon/collapsed95/**  
- **Type:** Genus-level OTU clustering  
- **OTU status:** ✅ YES (95% identity)  
- **Contents:** 258 OTU clusters (42.2% singletons)  
- **Biology:** Clusters sequences with ≥95% 16S identity (approximately genus-level resolution)  
- **Files:**  
  - `collapser_abundance_0.95.tsv` — OTU × sample abundance matrix  
  - `cluster_membership_map_0.95.tsv` — Original taxids → cluster_id mapping  
  - `cluster_centroids_0.95.fasta` — Representative sequences (FASTA)  
- **Columns:** `cluster_id`, `sample_id`, `abundance`, `read_count`, `member_taxids`, `member_taxa`, taxonomy  
- **Project role:** **PRIMARY OTU DATASET** — Most analyses use this resolution

**cannon/collapsed97/**  
- **Type:** Species-level OTU clustering  
- **OTU status:** ✅ YES (97% identity)  
- **Contents:** 353 OTU clusters (68.6% singletons)  
- **Biology:** Higher resolution clustering (species-level), more noise  
- **Project role:** Alternative resolution for species-specific questions

**cannon/collapsed95_per_sample/**, **cannon/collapsed97_per_sample/**  
- **Type:** Independent per-barcode clustering  
- **OTU status:** ✅ YES  
- **Contents:** Separate cluster_ids per sample  
- **Biology:** Useful for sample-specific diversity, testing clustering stability  
- **Project role:** Comparative mode validation

**cannon/CLUSTERING_COMPARISON.md**  
- **Type:** Analysis report  
- **Contents:** 95% vs 97% clustering comparison (singleton rates, cluster sizes, etc.)

---

### Filtered Outputs (oven/)

**Purpose:** Biologically-filtered OTU tables for downstream analysis

**oven/slashing/threepoint/**  
- **Type:** Reproducibility-filtered OTUs  
- **OTU status:** ✅ YES (95% clustering)  
- **Filter:** All 3 biological replicates present in ≥1 cohort  
- **Contents:** 169 OTU clusters (258→169)  
- **Biology:** **REPRODUCIBLE COMMUNITY MEMBERS** — Removes transient/contamination OTUs  
- **Rationale:** If OTU appears in all 3 fuzzsample reps OR all 3 drysample reps → consistent community member  
- **Project role:** Foundation for all downstream filtering

**oven/slashing/core_p2_mean0.5/**  
- **Type:** Core microbiome  
- **OTU status:** ✅ YES (95% clustering)  
- **Filter:** ≥2/3 replicates present + ≥0.5% mean relative abundance (within-cohort)  
- **Contents:** 58 OTU clusters (169→58)  
- **Biology:** **CORE MICROBIOME** — Prevalent AND abundant taxa (ecological dominance)  
- **Rationale:** Focuses on major community drivers (not just present, but numerically important)  
- **Project role:** Characterize dominant bacterial community

**oven/associated/**  
- **Type:** Degrader-associated OTUs  
- **OTU status:** ✅ YES (95% clustering)  
- **Filter:** Genus-level whitelist (14 target genera)  
- **Contents:** 18 OTU clusters (169→18)  
- **Biology:** **TARGET TAXA** — Dioxin/POP degrader-associated bacteria (Sphingomonas, Hephaestia, etc.)  
- **Genera:** 14 (from `refdbs/dioxin_pop_degrader_genera.txt`)  
- **Species:** 57 unique species collapsed into 18 OTUs  
- **Files:**  
  - `associated_OTU.tsv` — Wide format (samples × OTUs)  
  - `long_associated.tsv` — Long format for R (tidy data)  
  - `summary_associated.tsv` — Per-cluster statistics  
- **Project role:** **PRIMARY ANALYSIS TARGET** — Main biological question focus

**oven/slashing97/threepoint/**, **oven/slashing97/core_p2_mean0.5/**  
- **Type:** 97% clustering equivalents  
- **Contents:** 245 threepoint OTUs, 57 core OTUs  
- **Note:** Higher resolution, more noise

**oven/associated97/**  
- **Type:** 97% degrader filter attempt  
- **Contents:** **EMPTY** (0 OTUs)  
- **Reason:** Species-level clustering too stringent after threepoint filter  
- **Recommendation:** **Use 95% clustering for degrader analysis**

**oven/refined/**, **oven/brush/**  
- **Type:** Alternative analysis approaches  
- **Contents:** Species-level read-based analysis (brush), other experimental filters  
- **OTU status:** Varies (brush uses reads, not OTUs)

---

### R Compositional Analysis (R/)

**Purpose:** Statistical analysis and visualization

**R/aimed/**  
- **Type:** CLR-transformed degrader OTU analysis  
- **Input:** `oven/associated/long_associated.tsv` (18 OTUs)  
- **Contents:**  
  - `long_associated_clr.tsv` — Centered log-ratio transformed abundances  
  - `long_associated_clr_taxa.tsv` — CLR + full taxonomy  
  - `figures/*.png` — Heatmaps, boxplots (300 dpi)  
- **Biology:** **COMPOSITIONAL ANALYSIS** — Proper statistical treatment of relative abundance data  
- **Why CLR?** Handles zero-inflation, compositional constraints, enables valid statistical tests  
- **Project role:** Publication-ready degrader OTU figures

**R/outputs/pearson/**  
- **Type:** Correlation network analysis  
- **Input:** `oven/slashing/threepoint/` (169 OTUs)  
- **Contents:**  
  - `correlation_matrix.tsv` — Pearson correlations (OTU × OTU)  
  - `edges_r0.6.tsv` — Network edges (|r| ≥ 0.6)  
  - `figures/network_r0.6.png` — Network diagram  
- **Biology:** **CO-OCCURRENCE NETWORKS** — OTUs with similar abundance patterns across samples  
- **Interpretation:** Positive correlation = co-occurrence (shared niche), negative = competitive exclusion  
- **Project role:** Identify ecological guilds/functional groups

**R/nonassoc/**  
- **Type:** Non-degrader OTU analysis (QA)  
- **Contents:** Analysis of threepoint OTUs excluding degrader-associated taxa  
- **Project role:** Verify pipeline on non-target data

---

### Reference Databases (refdbs/)

**refdbs/mimt/**  
- **Type:** Emu taxonomic reference  
- **Contents:** MIMt 16S rRNA database (47k curated sequences)  
- **Features:** Low-redundancy species representatives, full-length 16S  
- **Build:** `scripts/build_mimt_emu.sh`  
- **Requires:** NCBI taxonomy files (`ncbi_tax/`)

**refdbs/dioxin_pop_degrader_genera.txt**  
- **Type:** Target genus whitelist  
- **Contents:** 14 genera (Sphingomonas, Hephaestia, Acetobacterium, etc.)  
- **Purpose:** Associated OTU filter input

---

### Utilities

**scripts/**  
- Core Python/Bash scripts (collapser_*, filter_*, cleaner.sh)

**workflows/emu-nf/**  
- Nextflow DSL2 pipeline for Emu classification  
- `bin/` contains helper scripts (format_emu_table.py, merge_emu_tables.py)

**components/metrics/**  
- Diversity analysis module (alpha/beta diversity calculations)

**awake/**  
- Lightweight CLR + network workflow (alternative to full R pipeline)

**reading/**  
- Literature references, filtering threshold citations

**cache/**, **work/**  
- Nextflow work directories (can be cleaned with `nextflow clean`)

---

## Key Workflows

### Workflow 1: QC Pipeline

**Script:** `scripts/cleaner.sh`

```bash
cd scripts/
bash cleaner.sh
```

**Parameters:**  
- `TOP_PERCENT=40` — Keep top 40% quality reads  
- `MIN_LEN=500` — Minimum read length  
- `MAX_LEN=2000` — Maximum read length

**Steps:**
1. Porechop: Remove ONT adapter sequences
2. Chopper: Filter by Q-score percentile and length
3. Output: `rin/filtered/*.fastq`

---

### Workflow 2: Emu Classification

**Script:** `workflows/emu-nf/main.nf`

```bash
cd workflows/emu-nf/
nextflow run main.nf \
  --reads ../../rin/filtered/*.fastq \
  --db ../../refdbs/mimt/emu_db \
  --outdir ../../rout/emu \
  --cpus 8
```

**Processes:**
1. `EMU_ABUNDANCE` — Per-sample classification with MIMt database
2. `MERGE_EMU_TABLES` — Combine into single abundance table

**Output:** `rout/emu/emu_abundance.tsv` (combined table, 436 taxa)

---

### Workflow 3: Collapser Table Build

**Script:** `scripts/collapser_build.py`

```bash
# Analysis track (min-reads 10)
python collapser_build.py \
  --emu-table ../rout/emu/emu_abundance.tsv \
  --min-reads 10 \
  --out ../rout/collapser/collapser_abundance.tsv

# Trace track (min-reads 1)
python collapser_build.py \
  --emu-table ../rout/emu/emu_abundance.tsv \
  --min-reads 1 \
  --out ../rout/collapser_trace/collapser_abundance.tsv
```

**Purpose:** Normalize Emu output, filter noise, prepare for clustering

---

### Workflow 4: OTU Clustering

**Script:** `scripts/collapser_cluster_0.95.sh`

```bash
cd scripts/
export OTU_ID=0.95  # or 0.97
bash collapser_cluster_0.95.sh
```

**Process:**
1. Extract observed taxids from collapser table
2. Retrieve 16S sequences from MIMt using seqkit
3. Cluster with vsearch (--cluster_fast, --id 0.95)
4. Map clusters back to original abundances
5. Output: `cannon/collapsed95/*.tsv`

**95% vs 97% guidance:**
- **95%:** Genus-level, less noise, recommended for most analyses
- **97%:** Species-level, higher resolution, more singletons

---

### Workflow 5: Threepoint Filter

**Script:** `scripts/filter_threepoint_core.py`

```bash
python filter_threepoint_core.py \
  ../cannon/collapsed95/collapser_abundance_0.95.tsv \
  ../oven/slashing/threepoint/
```

**Logic:** Keep OTUs present in ALL 3 biological replicates of ≥1 cohort

**Cohorts:**
- fuzzsample: barcodes 13, 14, 15
- drysample: barcodes 16, 17, 18

**Impact:** 258 → 169 clusters (removes 34.5%)

---

### Workflow 6: Core Filter

**Script:** `scripts/filter_core_otus.py`

```bash
python filter_core_otus.py \
  ../oven/slashing/threepoint/collapser_abundance_0.95_threepoint.tsv \
  ../oven/slashing/core_p2_mean0.5/ \
  --min-reps 2 \
  --min-mean-abundance 0.005
```

**Logic:** OTUs that are BOTH:
1. Prevalent: ≥2/3 replicates (within-cohort)
2. Abundant: ≥0.5% mean relative abundance

**Impact:** 169 → 58 clusters

---

### Workflow 7: Associated Filter

**Script:** `scripts/filter_associated_otus.py`

```bash
python filter_associated_otus.py \
  ../oven/slashing/threepoint/collapser_abundance_0.95_threepoint.tsv \
  ../refdbs/dioxin_pop_degrader_genera.txt \
  ../oven/associated/
```

**Logic:** Extract OTUs matching target genus whitelist

**Impact:** 169 → 18 clusters (14 genera, 57 species)

---

### Workflow 8: R Compositional Analysis

**Working directory:** `R/`

**Script sequence:**

```bash
cd R/scripts/

# 1. CLR transformation + heatmap
Rscript shep1_associated_clr.R
# Output: R/aimed/long_associated_clr.tsv, figures/clr_heatmap.png

# 2. Taxonomy join + boxplots
Rscript shep2_associated_clr_long_taxa.R
# Output: R/aimed/long_associated_clr_taxa.tsv, figures/genus_boxplots.png

# 3. Sample correlations (full threepoint dataset)
Rscript shep3_threepoint_sample_cor.R
# Output: R/outputs/pearson/correlation_matrix.tsv, edges_r0.6.tsv

# 4. Network visualization
Rscript shep4_threepoint_network.R
# Output: R/outputs/pearson/figures/network_r0.6.png
```

---

## Output Files Explained

### File Format 1: Emu Abundance Table

**Location:** `rout/emu/emu_abundance.tsv`

**Columns:**
- `sample_id` — Barcode identifier
- `tax_id` — NCBI taxonomy ID
- `estimated counts` — Read count
- `superkingdom`, `phylum`, `class`, `order`, `family`, `genus`, `species`

**Example:**
```tsv
sample_id	tax_id	estimated counts	superkingdom	phylum	...
barcode13_120_filtered	1090	125	Bacteria	Pseudomonadota	...
```

**Biology:** Each row = one species in one sample (no OTU clustering)

---

### File Format 2: OTU Abundance Table

**Location:** `cannon/collapsed95/collapser_abundance_0.95.tsv`

**Columns:**
- `cluster_id` — OTU identifier (e.g., cluster_001)
- `sample_id`
- `abundance` — Relative abundance (proportion)
- `read_count` — Total reads in this OTU
- `member_taxids` — Semicolon-separated taxids in this cluster
- `member_taxa` — Semicolon-separated species names
- Taxonomy columns

**Example:**
```tsv
cluster_id	sample_id	abundance	read_count	member_taxids	member_taxa	...
cluster_001	barcode13_120_filtered	0.0523	125	1090;287	Pseudomonas aeruginosa;Pseudomonas fluorescens	...
```

**Biology:** Each row = one OTU in one sample. Multiple species may collapse into one OTU.

---

### File Format 3: Cluster Membership Map

**Location:** `cannon/collapsed95/cluster_membership_map_0.95.tsv`

**Columns:**
- `taxid` — Original NCBI taxonomy ID
- `cluster_id` — Assigned OTU cluster

**Example:**
```tsv
taxid	cluster_id
1090	cluster_001
287	cluster_001
470	cluster_002
```

**Purpose:** Trace which species contribute to each OTU

---

### File Format 4: Long Format (R-ready)

**Location:** `oven/associated/long_associated.tsv`

**Columns:**
- `cluster_id`
- `sample_id`
- `estimated_counts` — Abundance (absolute counts)
- Full taxonomy columns

**Purpose:** Tidy format for R/ggplot2

---

### File Format 5: CLR-Transformed

**Location:** `R/aimed/long_associated_clr.tsv`

**Columns:**
- `cluster_id`
- `sample_id`
- `clr_value` — Centered log-ratio

**Purpose:** Compositionally-appropriate values for statistics

---

### File Format 6: Network Edges

**Location:** `R/outputs/pearson/edges_r0.6.tsv`

**Columns:**
- `source_cluster`
- `target_cluster`
- `correlation` — Pearson r
- `p_value`

**Purpose:** igraph network edges

---

## Targeted Gene Integration

### Overview: Linking 16S Community Data to Functional Genes

The [playground targeted gene analysis](../playground/) provides **orthogonal validation** of dioxin degradation capacity detected through 16S community profiling. This section explains how the two datasets integrate.

### Targeted Gene Pipeline Summary

**Location:** [../playground/vork_clean/](../playground/vork_clean/)

**Approach:** PCR amplification + ONT sequencing + Medaka consensus calling

**Samples:** 2 soil DNA extracts (SL-1A, SL-2A) across 4 barcodes (BC01-04)
- BC01-02: SL-1A technical replicates
- BC03-04: SL-2A technical replicates

**Target Genes:**
1. **clcA** — Chlorocatechol-1,2-dioxygenase (ring cleavage enzyme)
2. **catB2** — Muconate cycloisomerase (β-ketoadipate pathway)
3. **BpHc** — Biphenyl pathway component (PCB degradation)
4. **ntDAa** — Naphthalene dioxygenase (PAH degradation)

### Key Results from Targeted Sequencing

| Gene | Reads Detected | Consensus Seqs | Identity to Refs | Taxonomic Assignment |
|------|----------------|----------------|------------------|----------------------|
| **clcA** | 11,923 (78%) | 7 | 99.1-99.4% | *Pseudomonas aeruginosa*, *Diaphorobacter* sp. |
| **catB2** | 2,521 (19%) | 6 | 87.7-90.5% | *Sphingobium yanoikuyae* |
| **BpHc** | 30 (0.2%) | 0 | N/A | Insufficient depth (likely primer failure) |
| **ntDAa** | 0 | 0 | N/A | Complete failure (primers ineffective) |

**Consensus quality:** 13/13 sequences (100%) passed QC with coverage 14.8×–1502× and <1% ambiguous bases.

**Multiple sequence alignment:** clcA sequences aligned to 576 bp conserved core (49.4% gap reduction from full-length alignment).

### Taxa-to-Function Linkage

**Integration approach:** Match genus-level taxonomy between 16S OTUs and targeted gene consensus assignments.

#### Evidence for *Sphingomonas*/*Sphingobium* (catB2 carriers)
- **16S detection:** 8 OTUs matched *Sphingomonas*, present in all 6 samples (100% detection)
- **Targeted gene:** 6 catB2 consensus sequences from *Sphingobium yanoikuyae*
- **Interpretation:** *Sphingomonadaceae* family members detected in 16S carry catB2 muconate cycloisomerase gene
- **Functional capacity:** Supported (β-ketoadipate pathway enzyme confirmed)

#### Evidence for *Pseudomonas*/*Diaphorobacter* (clcA carriers)
- **16S detection:** Multiple OTUs matched *Pseudomonas* and *Burkholderia* (Betaproteobacteria)
- **Targeted gene:** 7 clcA consensus sequences from *Pseudomonas aeruginosa* and *Diaphorobacter* sp.
- **Interpretation:** Pseudomonadota taxa detected in 16S carry clcA ring cleavage enzyme
- **Functional capacity:** Supported (chlorocatechol degradation confirmed)

#### Evidence for *Desulfitobacterium* (reductive dechlorination)
- **16S detection:** 2 OTUs matched *Desulfitobacterium*, present in all 6 samples
- **Targeted gene:** No direct gene target (rdh genes not included in primer panel)
- **Interpretation:** 16S suggests reductive dechlorination capacity, but functional genes not validated
- **Functional capacity:** Inferred from taxonomy only (requires verification)
Different sampling events:**
- 16S barcodes: BC13-18 (fuzzsample, drysample) — Six soil samples
- Targeted gene barcodes: BC01-04 (SL-1A, SL-2A) — Two soil extracts (different sampling)
- **Mapping status:** CONFIRMED as separate samples — treat as two independent approaches to the same site
- **Consequence:** Cannot link at sample level (e.g., "BC13 has clcA from Pseudomonas")
- **Valid integration:** Genus-level linkage only (e.g., "*Sphingomonas* detected in 16S, catB2 confirmed in targeted sequencing
- **Mapping status:** UNCONFIRMED — no explicit sample linking file found
- **Consequence:** Cannot link at sample level (e.g., "BC13 has clcA from Pseudomonas")
- **Valid integration:** Genus-level linkage only (e.g., "*Sphingomonas* detected in 16S, catB2 confirmed in playground")

**⚠️ Taxonomy ≠ function:**
- Genus-level matching doesn't prove the 16S-detected OTU carries the gene
- Example: *Sphingomonas* OTU from 16S may be a different strain than *Sphingobium* catB2 carrier
- Functional genes are often horizontally transferred—phylogeny is not proof

**⚠️ Primer bias:**
- BpHc and ntDAa failures indicate primer design issues
- Absence of signal ≠ absence of genes (false negatives possible)
- Only 2/4 gene targets successfully amplified

### Recommended Integration Strategy

**For thesis/publication:**
1. **Valid claim:** "16S detected *Sphingomonas*; targeted sequencing confirmed catB2 genes from *Sphingobium*"
2. **Invalid claim:** "Sample BC13 contains *Sphingomonas* with catB2 genes" (unless sample mapping confirmed)
3. **Best practice:** Report as **taxa-to-function linkage at genus level**, not sample-specific proof

**Data sources:**
- 16S OTU tables: [oven/associated/](oven/associated/)
- Targeted gene consensus: [../playground/vork_clean/roadtrip/results/medaka/all_barcodes_consensus.fasta](../playground/vork_clean/roadtrip/results/medaka/all_barcodes_consensus.fasta)
- Alignment: [../playground/alignment/trimmed_alignments/clca_trimmed.fasta](../playground/alignment/trimmed_alignments/clca_trimmed.fasta)

---

## Methodological Notes & Critical Assessment

### OTU vs EMU: What's the Difference?

This is a **critical distinction** that affects how you interpret diversity metrics.

#### EMU (Pre-OTU) Metrics
- **What it is:** Direct taxonomic classification from MIMt 16S database
- **How it works:** Each read matched to closest reference species (many-to-many mapping)
- **Output:** 437 unique taxids, ~250 genera
- **Diversity:** Shannon 3.58–4.55 (genus-level)
- **Use case:** Taxonomic richness, species-level presence/absence

#### OTU (Post-Clustering) Metrics
- **What it is:** Operational taxonomic units from sequence similarity clustering
- **How it works:** Reads clustered at 95%/97% identity (many reads → one OTU)
- **Output:** 258 OTU95 clusters (or 353 OTU97)
- **Diversity:** Shannon 2.52–2.96 (OTU97)
- **Use case:** Ecological inference, community structure, statistical comparisons

**Why are OTU diversity values lower?** Clustering collapses similar taxa into single units. Example: 8 *Sphingomonas* species → 1 OTU95 cluster.

**Which should I report?** **Both**, clearly labeled. EMU shows taxonomic breadth; OTU shows ecological diversity.

### 95% vs 97% Clustering: Which Should I Use?

| Criterion | OTU95 (Genus-level) | OTU97 (Species-level) ✅ |
|-----------|----------------------|----------------------|
| **Biological interpretation** | Approximates genus-level | Approximates species-level |
| **Singleton rate** | 42% | 69% |
| **Noise level** | Lower (more robust) | Higher (sensitive to errors) |
| **Recommended for** | Initial community profiling, broad trends | **Degrader analysis, post-taxonomy filtering** |

**Analytical Strategy:**
- **Step 1:** Use **EMU abundance table** for raw taxonomic findings (437 taxids detected)
- **Step 2:** Use **OTU95** for genus-level community trends (phylum composition, diversity)
- **Step 3:** Use **OTU97** for species-level analysis of **degrader-associated taxa** (a priori target list)

**Rationale for multi-resolution approach:**
- EMU provides maximum taxonomic detail (pre-clustering)
- OTU95 reduces noise for broad ecological patterns
- OTU97 provides species-level discrimination for focused degrader analysis after taxonomy filtering

**Key insight:** Post-taxonomy filtering (associated filter) removes spurious OTU97 singletons, making species-level resolution viable for downstream functional analysis.

### Threepoint vs Core Filters: When to Use Each?

| Filter | Logic | Retention | Purpose | Use Case |
|--------|-------|-----------|---------|----------|
| **Threepoint** | All 3 reps in ≥1 cohort | 192/258 (74%) | Reproducibility | Exploratory analyses, cohort comparisons |
| **Core** | ≥2/3 reps + ≥0.5% mean | 55/192 (29%) | Dominance | Focus on numerically important taxa |
| **Associated** | Genus whitelist | 18/192 (9%) | Functional targeting | Degrader-specific questions |

**Recommendation:** Start with **threepoint** (removes transient taxa). Apply **core** for dominant community drivers. Use **associated** only for hypothesis-driven degrader analysis.

### Small Sample Size (n=3 per cohort): Limitations

**Statistical power:** Minimal—cannot reliably detect effects <2 standard deviations

**Inference:** Treat all cohort comparisons as **exploratory**, not confirmatory

**Reporting guidelines:**
- ✅ Report effect sizes and confidence intervals
- ✅ Emphasize descriptive patterns
- ❌ Avoid strong causal claims
- ❌ Don't rely solely on p-values (risk of false positives/negatives)

**Mitigation strategy:** Use reproducibility (threepoint filter) as evidence of biological signal, not just statistical significance.

### Compositional Data: Why CLR Transformation?

**Problem:** Relative abundance data are compositional (sum to 100%)—standard statistics are invalid.

**Solution:** Centered log-ratio (CLR) transformation converts relative abundances to real-valued space.

**Effect:** Correlations on CLR values are valid; correlations on raw counts are spurious.

**Application in this pipeline:**
- Network analysis: Correlations computed on CLR-transformed abundances ([R/aimed/long_associated_clr.tsv](R/aimed/long_associated_clr.tsv))
- DO NOT correlate raw OTU counts from [oven/associated/associated_OTU.tsv](oven/associated/associated_OTU.tsv)

**Reference:** Gloor et al. (2017) Front Microbiol 8:2224. [DOI:10.3389/fmicb.2017.02224](https://doi.org/10.3389/fmicb.2017.02224)

### Co-Occurrence Networks: Interpretation Caveats

**What networks show:** Statistical associations (correlations) in CLR space

**What networks DO NOT show:**
- ❌ Ecological interactions (competition, mutualism)
- ❌ Causation (A → B)
- ❌ Metabolic dependencies

**Why?** n=6 samples provide insufficient power to distinguish spurious from real correlations.

**Appropriate use:** Hypothesis generation, exploratory pattern detection, sensitivity analysis across thresholds.

**Inappropriate use:** Claiming "Sphingomonas co-occurs with Desulfitobacterium, therefore they interact."

### Degrader-Associated Taxa: Taxonomy ≠ Function

**Selection method:** Genus-level matching to predefined whitelist ([refdbs/dioxin_pop_degrader_genera.txt](refdbs/dioxin_pop_degrader_genera.txt))

**What this proves:** Taxa with genus names matching literature-cited degraders are present

**What this DOES NOT prove:**
- ❌ The detected strain has degradation capacity
- ❌ Degradation genes are expressed
- ❌ Genes are functional (not pseudogenes)

**Validation required:** Targeted gene sequencing (playground) provides partial validation for *Sphingomonas* (catB2) and *Pseudomonas* (clcA). For other genera (*Desulfitobacterium*, *Acetobacterium*), functional capacity is **inferred only**.

### Data Provenance & Reproducibility

All quantitative claims are traceable to:
- **JSON summaries:** [../writeup/triage_summaries/sixteen_*.summary.json](../writeup/triage_summaries/)
- **OTU tables:** [oven/slashing/threepoint/](oven/slashing/threepoint/), [oven/slashing/core_p2_mean0.5/](oven/slashing/core_p2_mean0.5/), [oven/associated/](oven/associated/)
- **Diversity files:** [../writeup/assist/sixteen_otu97_base_alpha_diversity.tsv](../writeup/assist/sixteen_otu97_base_alpha_diversity.tsv)
- **Phylum composition:** [components/output/phylum_cohort_summary.csv](components/output/phylum_cohort_summary.csv)

**No claims should be made without traceable evidence to these files.**

---

## Figures & Tables Reference

### Thesis-Ready Figures (Publication Quality, 300 DPI)

#### Alpha Diversity
- **File:** [components/output/fig2_alpha_diversity.png](components/output/fig2_alpha_diversity.png)
- **Content:** Shannon/Simpson indices by sample and cohort
- **Source data:** components analysis on EMU abundance table
- **Caption template:** "Alpha diversity metrics (Shannon index) for bacterial communities in fuzzsample (BC13-15) and drysample (BC16-18) cohorts, calculated from EMU genus-level classifications."

#### Beta Diversity (PCoA Ordination)
- **File:** [components/output/fig3_beta_diversity_pcoa.png](components/output/fig3_beta_diversity_pcoa.png)
- **Content:** Principal coordinates analysis with cohort coloring
- **Source data:** Bray-Curtis dissimilarities from EMU or OTU table
- **Caption template:** "Principal coordinates analysis (PCoA) of bacterial community composition showing cohort separation based on Bray-Curtis dissimilarities."

#### Phylum Composition (Per-Sample)
- **File:** [components/output/phylum_top3_stacked_per_sample.png](components/output/phylum_top3_stacked_per_sample.png)
- **Content:** Stacked bar chart of top 3 phyla by sample
- **Source data:** [components/output/phylum_per_sample_data.csv](components/output/phylum_per_sample_data.csv)
- **Caption template:** "Relative abundances of the three most abundant phyla (Bacillota, Pseudomonadota, Bacteroidota) across six soil samples. Bacillota dominate fuzzsample; Bacteroidota are enriched in drysample."

#### Phylum Composition (Cohort Means)
- **File:** [components/output/phylum_top3_cohort_mean_sd.png](components/output/phylum_top3_cohort_mean_sd.png)
- **Content:** Grouped bar chart with error bars (mean ± SD)
- **Source data:** [components/output/phylum_cohort_summary.csv](components/output/phylum_cohort_summary.csv)
- **Caption template:** "Cohort-level mean relative abundances (± standard deviation) of top 3 phyla. Bacteroidota show 7.6-fold depletion in fuzzsample (1.1% vs 8.4%)."

#### CLR Heatmap (Degrader OTUs)
- **File:** [R/aimed/figures/fig_associated_clr_heatmap.png](R/aimed/figures/fig_associated_clr_heatmap.png)
- **Content:** Heatmap of 18 degrader-associated OTUs × 6 samples (CLR-transformed)
- **Source data:** [R/aimed/long_associated_clr.tsv](R/aimed/long_associated_clr.tsv)
- **Caption template:** "CLR-transformed abundances of degrader-associated OTU95 clusters across samples. Sphingomonas-related OTUs show consistent presence; Desulfitobacterium varies by cohort."

#### Genus Boxplots (Degrader OTUs)
- **File:** [R/aimed/figures/fig_associated_genus_clr_boxplot.png](R/aimed/figures/fig_associated_genus_clr_boxplot.png)
- **Content:** Boxplots of CLR values by genus and cohort
- **Source data:** [R/aimed/long_associated_clr_long_with_taxa.tsv](R/aimed/long_associated_clr_long_with_taxa.tsv)
- **Caption template:** "Distribution of CLR-transformed abundances for degrader-associated genera by cohort. Sphingomonas and Novosphingobium show high consistency; Desulfitobacterium varies."

#### Co-Occurrence Network
- **File:** [R/aimed/figures/fig_associated_louvain_r0.60.png](R/aimed/figures/fig_associated_louvain_r0.60.png)
- **Content:** Network graph (nodes = OTUs, edges = correlations ≥0.6 on CLR values)
- **Source data:** [R/aimed/network/edges_r0.60.tsv](R/aimed/network/edges_r0.60.tsv)
- **Caption template:** "Exploratory co-occurrence network of degrader-associated OTUs (Pearson r ≥ 0.6 on CLR values). Communities detected via Louvain algorithm. Network is hypothesis-generating only due to small sample size (n=6)."

### Backing Data Tables

#### Threepoint OTU95 Summary
- **File:** [../writeup/triage_summaries/sixteen_otu95_threepoint.summary.json](../writeup/triage_summaries/sixteen_otu95_threepoint.summary.json)
- **Content:** 192 OTUs, 6 samples, sparsity, top OTUs by mean/prevalence
- **Use:** Cite for "192 reproducible OTU95 clusters" claim

#### Core OTU95 Summary
- **File:** [../writeup/triage_summaries/sixteen_otu95_core.summary.json](../writeup/triage_summaries/sixteen_otu95_core.summary.json)
- **Content:** 55 OTUs, 6 samples, core microbiome definition
- **Use:** Cite for "55 core OTUs" claim

#### Associated OTU95 Summary
- **File:** [../writeup/triage_summaries/sixteen_associated_otu95.summary.json](../writeup/triage_summaries/sixteen_associated_otu95.summary.json)
- **Content:** 18 OTUs, 14 genera, degrader whitelist matches
- **Use:** Cite for "18 degrader-associated OTUs from 14 genera" claim

#### Phylum Composition Statistics
- **File:** [components/output/phylum_statistics.csv](components/output/phylum_statistics.csv)
- **Content:** All 19 detected phyla with per-sample and cohort-level statistics
- **Use:** Cite for phylum-level abundance values

#### Alpha Diversity (OTU97)
- **File:** [../writeup/assist/sixteen_otu97_base_alpha_diversity.tsv](../writeup/assist/sixteen_otu97_base_alpha_diversity.tsv)
- **Content:** Shannon/Simpson indices for OTU97 clustering
- **Use:** Cite for OTU-level diversity values (Shannon 2.52–2.96)

### Figure Generation Scripts

All figures can be regenerated from source data:

```bash
# Phylum composition figures
cd components/
python analyze_phylum_composition.py --input ../cannon/emu/emu_abundance.tsv --output output --top-n 3

# CLR heatmaps and networks (R)
cd ../R/scripts/
Rscript shep1_associated_clr.R
Rscript shep2_associated_clr_long_taxa.R
Rscript shep4_threepoint_network.R
```

**Documentation:** See [components/output/QUICK_REFERENCE.md](components/output/QUICK_REFERENCE.md) for copy-paste ready results paragraphs.

---

## Usage Examples

### Example 1: Process New Samples

```bash
# 1. Add new FASTQs
cp /path/to/barcode19.fastq rin/hq/

# 2. Run QC
cd scripts/ && bash cleaner.sh

# 3. Run Emu (new barcode only)
cd ../workflows/emu-nf/
nextflow run main.nf --reads ../../rin/filtered/barcode19*.fastq --db ../../refdbs/mimt/emu_db

# 4. Rebuild collapser with ALL samples
cd ../../scripts/
python collapser_build.py --emu-table ../rout/emu/emu_abundance.tsv --min-reads 10 --out ../rout/collapser/collapser_abundance.tsv

# 5. Re-run OTU clustering
bash collapser_cluster_0.95.sh

# 6. Re-run filters
python filter_threepoint_core.py ../cannon/collapsed95/collapser_abundance_0.95.tsv ../oven/slashing/threepoint/
python filter_core_otus.py ../oven/slashing/threepoint/collapser_abundance_0.95_threepoint.tsv ../oven/slashing/core_p2_mean0.5/
python filter_associated_otus.py ../oven/slashing/threepoint/collapser_abundance_0.95_threepoint.tsv ../refdbs/dioxin_pop_degrader_genera.txt ../oven/associated/
```

---

### Example 2: Custom Genus Extraction

```bash
# Extract Pseudomonas OTUs only
echo "Pseudomonas" > /tmp/pseudomonas_only.txt

python filter_associated_otus.py \
  ../oven/slashing/threepoint/collapser_abundance_0.95_threepoint.tsv \
  /tmp/pseudomonas_only.txt \
  ../oven/pseudomonas_subset/
```

---

### Example 3: Ultra-Core Filter

```bash
# All 3 reps + ≥1% abundance (more stringent than default)
python filter_core_otus.py \
  ../oven/slashing/threepoint/collapser_abundance_0.95_threepoint.tsv \
  ../oven/slashing/ultracore/ \
  --min-reps 3 \
  --min-mean-abundance 0.01
```

---

## Troubleshooting

### Issue: High Singleton Rate

**Symptom:** 40-70% of OTUs are singletons

**Solutions:**
1. Use 95% clustering instead of 97%
2. Apply threepoint filter
3. Increase min-reads threshold (e.g., 20)

---

### Issue: Zero Associated OTUs at 97%

**Symptom:** `oven/associated97/` is empty

**Cause:** 97% clustering creates too many low-prevalence clusters

**Solutions:**
1. **Use 95% clustering** (recommended)
2. Skip threepoint filter at 97%
3. Relax prevalence threshold

---

### Issue: Environment File Missing

**Symptom:** `env/sixteen.yml` not found

**Solution:**
```bash
cp ../sixtran/sixteen.yml env/sixteen.yml
```

---

### Issue: Emu Database Not Found

**Solution:**
```bash
cd refdbs/mimt/
bash scripts/build_mimt_emu.sh
```

---

### Issue: Nextflow Work Directory Too Large

**Solution:**
```bash
cd workflows/emu-nf/
nextflow clean -f
```

---

## References

### Related Documentation

- [primer.md](primer.md) — Detailed operator guide
- [where-we-are.md](where-we-are.md) — Project status (Jan 2026)
- [DEV_CHECKLIST.md](DEV_CHECKLIST.md) — Maintenance tasks
- [R/README.md](R/README.md) — Stage 3 analysis docs
- [cannon/CLUSTERING_COMPARISON.md](cannon/CLUSTERING_COMPARISON.md) — 95% vs 97% comparison
- [reading/README.md](reading/README.md) — Literature references

---

## Project Status

**Last updated:** January 8, 2026

**Current analysis:**
- 6 samples processed (barcodes 13-18)
- 258 OTU clusters at 95% (169 after threepoint, 58 core, 18 associated)
- 14 target degrader genera identified
- Compositional analysis complete

**Known limitations:**
- Small sample size (n=3/cohort) limits statistical power
- 97% clustering too stringent for degrader analysis
- Singleton rate high (42% @ 95%, 69% @ 97%)

**Next steps:**
- Process additional samples
- Validate degrader genera with functional assays
- Phylogenetic placement of novel OTUs

---

**Cleaned up:** Removed deprecated `oven/painted/`, `R/painted/`, empty `vendor/`, `slasher/` directories
