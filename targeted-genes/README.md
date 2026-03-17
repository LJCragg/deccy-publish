# roadtrip

**Targeted functional gene amplicon pipeline for Oxford Nanopore Technologies (ONT) data**

Detects and characterises dioxin-degradation genes (clcA, catB2, BpHc, ntDAa) in anonymized site contaminated soils using PCR amplification + ONT sequencing + Medaka consensus polishing.

**Last updated:** 2026-03-17
**Current dataset:** 4 barcodes (BC01–BC04) representing 2 soil DNA extracts × 2 technical replicates each

---

## Overview

This pipeline applies targeted PCR amplification of four functional dioxin-degradation genes followed by ONT long-read sequencing, read clustering, and consensus sequence generation. It complements the community-level 16S and fungal pipelines by providing direct sequence evidence of degradation pathway enzymes present in anonymized site soil DNA.

The pipeline is implemented as a Snakemake workflow with modular components (rules in `rules/` and `components/`).

---

## Project Context

The anonymized contaminated site in anonymized New Zealand site, New Zealand was contaminated with **2,3,7,8-Tetrachlorodibenzo-p-dioxin (TCDD)** and related chlorophenols as byproducts of herbicide production by anonymized contaminated site between **1962 and 1987**. This study employs three complementary sequencing strategies:

1. **16S rRNA bacterial amplicon** — community-level taxonomic profiling ([../sixteen/README.md](../sixteen/README.md))
2. **Fungal ITS/18S amplicon** — fungal community structure and POP-degrader guilds ([../funcall/README.md](../funcall/README.md))
3. **Targeted functional gene sequencing** (THIS PIPELINE) — direct detection of degradation genes

---

## Samples

| Barcode | Sample | Description |
|---------|--------|-------------|
| barcode01 | fuzzsample_rep1 | Wet soil DNA, technical replicate 1 |
| barcode02 | fuzzsample_rep2 | Wet soil DNA, technical replicate 2 |
| barcode03 | drysample_rep1 | Dry soil DNA, technical replicate 1 |
| barcode04 | drysample_rep2 | Dry soil DNA, technical replicate 2 |
| unclassified | — | Mixed/ambiguous barcode assignments |

Two distinct soil cohorts (fuzzsample, drysample) each sequenced in duplicate to assess technical reproducibility.

---

## Target Genes

| Gene | Pathway | Enzyme Function | Outcome |
|------|---------|-----------------|---------|
| **clcA** | Chlorocatechol degradation | Chlorocatechol 1,2-dioxygenase | **Confirmed** — 11,923 reads, 7 consensus (98.2–99.4% identity) |
| **catB2** | Catechol degradation | Muconate cycloisomerase | **Confirmed** — 2,521 reads, 6 consensus (87.7–90.5% identity) |
| **BpHc** | Biphenyl degradation | Biphenyl dioxygenase | **Detected (marginal)** — 30 reads, below confidence threshold |
| **ntDAa** | Naphthalene degradation | Naphthalene dioxygenase | **Not recovered** — 0 reads; amplification failure |

---

## Pipeline Stages

### Stage 1: Slashing (QC Filtering)
**Tools:** Porechop (adapter trimming) + Chopper (quality/length filtering)
**Rule:** `rules/slashing.smk`
**Parameters:** Q≥10, length 300–2000 bp, headcrop/tailcrop 15 bp

### Stage 2: Greedhunt (Target Mapping)
**Tool:** Minimap2 — maps QC'd reads against 4 target gene references
**Rule:** `components/greedhunt/rules/greedhunt.smk`
**Output:** Per-gene FASTQ files extracted from mapped reads
**Result:** 14,474 reads assigned (12.2% of post-filter reads)

### Stage 3: IsONclust3 (Read Clustering)
**Tool:** IsONclust3 — ONT-aware read clustering
**Rule:** `components/isonclust3/rules/isonclust3.smk`
**Output:** Per-cluster FASTQ files
**Result:** 48 total clusters across all barcodes and genes

### Stage 4: Medaka (Consensus Polishing)
**Tool:** Medaka — neural-network consensus caller for ONT reads
**Rule:** `components/medaka/rules/medaka.smk`
**Strategy:** Top-N reads per cluster submitted to Medaka polishing
**Result:** 13 high-quality consensus sequences (7 clcA, 6 catB2)

### Stage 5: Postage (Abundance Table)
**Tool:** Custom Minimap2 + idxstats wrapper
**Rule:** `components/postage/rules/map.smk`
**Output:** Per-sample abundance table with coverage depth headers
**Result:** Read counts and coverage statistics per consensus sequence

### Stage 6: MAFFT (Multiple Sequence Alignment)
**Tool:** MAFFT — multiple sequence alignment
**Rule:** `components/mafft/rules/mafft.smk`
**Output:** Aligned FASTA of consensus sequences
**Result:** 576 bp conserved core region identified for clcA (22.79% gap rate)

---

## Data Layout

Large inputs and outputs are stored under `data/` (gitignored) and symlinked in:

| Symlink | Target | Contents |
|---------|--------|---------|
| `forme` | `../data/roadtrip/forme` | Canonical FASTQ inputs (raw/demuxed reads) |
| `results` | `../data/roadtrip/results` | Canonical pipeline outputs |

Tracked outputs also written to `outputs/` in this directory.

---

## Tracked Outputs

`outputs/` contains:
- `alignment_notes/` — MAFFT alignment reports and summaries
- Per-run QC summaries (retention stats by barcode)
- Consensus inventory (sequence IDs, identity scores, top BLAST hits)

The `results` symlink (→ `data/roadtrip/results`) holds full intermediates and large files.

---

## Key Results

| Metric | Value |
|--------|-------|
| Raw reads (4 barcodes) | 308,806 |
| Raw reads incl. unclassified | 381,589 |
| Post-filter reads | 115,660 (37.5% retention) |
| Assigned to targets | 14,474 (12.2%) |
| clcA reads | 11,923 (82.4% of assigned) |
| catB2 reads | 2,521 (17.4%) |
| BpHc reads | 30 (0.2%, below threshold) |
| ntDAa reads | 0 (amplification failure) |
| Total clusters | 48 |
| Consensus sequences | 13 |
| clcA consensus | 7 (98.2–99.4% to reference) |
| catB2 consensus | 6 (87.7–90.5% to reference) |
| clcA conserved alignment | 576 bp, 22.79% gaps |

---

## Running the Pipeline

```bash
cd roadtrip
conda activate roadtrip   # or: mamba activate roadtrip

# Full pipeline
snakemake --use-conda -j8

# Dry run first
snakemake --use-conda -j8 -n

# Resume from checkpoint
snakemake --use-conda -j8 --rerun-incomplete
```

Configuration: `config/roadtrip.yml` — set barcode list, target gene paths, Medaka model.

---

## Related

- [../sixteen/README.md](../sixteen/README.md) — 16S bacterial community analysis
- [../funcall/README.md](../funcall/README.md) — Fungal ITS/18S community analysis
- [../waffle/targeted_genes/](../waffle/targeted_genes/) — Results section rewrite (active)
- [../waffle/targeted_genes/00_plan.md](../waffle/targeted_genes/00_plan.md) — Writing plan with verified numbers
