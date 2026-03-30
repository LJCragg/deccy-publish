# Targeted Functional Gene Analysis

**Targeted functional gene amplicon analysis of dioxin-degradation genes using Oxford Nanopore Technologies (ONT)**

Detects and characterises four functional genes (clcA, catB2, BpHc, ntDAa) in TCDD-contaminated soil DNA using PCR amplification, ONT sequencing, and Medaka consensus polishing.

**Last updated:** 2026-03-17
**Dataset:** 4 barcodes (BC01–BC04) — 2 soil DNA extracts (SS1, SS2) × 2 technical replicates each

---

## Samples

| Barcode | Sample | Description |
|---------|--------|-------------|
| barcode01 | SS1_rep1 | SS1 soil DNA, technical replicate 1 |
| barcode02 | SS1_rep2 | SS1 soil DNA, technical replicate 2 |
| barcode03 | SS2_rep1 | SS2 soil DNA, technical replicate 1 |
| barcode04 | SS2_rep2 | SS2 soil DNA, technical replicate 2 |
| unclassified | — | Mixed/ambiguous barcode assignments |

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

| Stage | Tool | Purpose |
|-------|------|---------|
| QC filtering | Porechop + Chopper | Adapter trimming; Q≥10, 300–2000 bp |
| Target mapping | Minimap2 | Maps QC reads against 4 gene references |
| Read clustering | IsONclust3 | ONT-aware read clustering |
| Consensus polishing | Medaka | Neural-network consensus from top reads per cluster |
| Abundance table | Minimap2 + idxstats | Read counts and coverage depth per consensus |
| Sequence alignment | MAFFT | Multiple sequence alignment of consensus sequences |

---

## Key Results

| Metric | Value |
|--------|-------|
| Raw reads (4 barcodes) | 308,806 |
| Raw reads including unclassified | 381,589 |
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

## Tracked Outputs

| Directory | Contents |
|-----------|----------|
| `outputs/greedhunt/` | Per-barcode mapping statistics (JSON, TXT) |
| `outputs/mafft/` | Aligned FASTA files per target gene |
| `outputs/medaka/` | All consensus sequences (FASTA) |
| `outputs/postage/tables/` | Per-barcode abundance tables (TSV) |
| `outputs/postage/coverage/` | Per-barcode coverage depth files |
| `outputs/slashing_qc/` | QC filtering summary report |
| `outputs/alignment_notes/` | clcA trimmed alignment FASTA; alignment comparison report |

---

## Pipeline Configuration

Pipeline implemented as a Snakemake workflow. Configuration files are in `config/` and environment specifications in `envs/`. Component manifests are in `components/`.

Large inputs and outputs are stored under `data/` (gitignored) and symlinked in:

| Symlink | Contents |
|---------|---------|
| `forme` | Canonical FASTQ inputs (demultiplexed reads) |
| `results` | Full pipeline intermediates and outputs |
