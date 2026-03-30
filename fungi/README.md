# Fungal ITS/18S Amplicon Analysis

**Fungal community profiling from ITS/18S amplicons (Oxford Nanopore Technologies)**

Taxonomic classification and OTU-level clustering of fungal communities, including identification of POP-degrader associated taxa and plant-associated fungi.

---

## Samples

| Barcode | Description |
|---------|-------------|
| barcode19 | Fungal ITS/18S amplicon library 1 |
| barcode20 | Fungal ITS/18S amplicon library 2 |
| barcode21 | Fungal ITS/18S amplicon library 3 |

**Sequencing:** ONT MinION. Basecalling: Dorado 1.3.0, HAC model (`dna_r10.4.1_e8.2_400bps_hac@v4.3.0`).

---

## Key Results

| Metric | Value |
|--------|-------|
| Raw reads | 849,683 |
| Post-QC reads | 359,844 (~42% retention) |
| Total taxa detected (EMU) | 1,306 |
| Fungal entries | 876 (67.1% of total) |
| Non-fungal entries | 429 (plants, animals, algae, oomycetes) |
| OTU95 clusters (fungi-only, pre-filter) | 83 |
| OTU95 clusters after prevalence filter | 91 |
| Degrader genera detected | 15 of 51 queried |

**QC parameters:** Porechop (adapter trimming) + Chopper (Q≥15, 1000–7000 bp, 15 bp end-crop).

**Taxonomic classification:** EMU against MIMt 18S+ITS database (61,924 sequences; NCBI taxonomy). Alternative: UNITE ITS (100,176 sequences).

**OTU clustering:** VSEARCH 95% identity (genus-level resolution). Prevalence filter: ≥10 reads in all 3 replicates.

**Fungi-only analysis:** Non-fungal eukaryotes (plants, animals, nematodes) removed at read level prior to OTU clustering. Chlorophyta retained.

---

## Directory Structure

| Directory | Contents |
|-----------|----------|
| `emure/` | Reference-based OTU95 clustering — prefilter tables, collapser input, reference sequences, cluster assignments, OTU tables, prevalence-filtered outputs |
| `pukeko/` | Fungi-only filtered OTU analysis — EMU abundance (fungal), OTU95 tables, diversity metrics |
| `house/` | Basecalling documentation |
| `mapping/` | EMU classification documentation |
| `env/` | Conda environment specification (`funcall.yml`) |

Thesis figures and statistical analysis tables are in [`../r_analysis/`](../r_analysis/).

---

## Per-Sample Read Counts

| Barcode | Raw reads | Post-QC reads | Retention | Mean length (pre-QC) |
|---------|-----------|---------------|-----------|----------------------|
| barcode19 | 101,944 | 43,043 | 42.2% | 5,589 bp |
| barcode20 | 248,391 | 104,959 | 42.3% | 5,316 bp |
| barcode21 | 433,973 | 192,864 | 44.4% | 4,998 bp |
| unclassified | 65,375 | 18,978 | 29.0% | 5,202 bp |
