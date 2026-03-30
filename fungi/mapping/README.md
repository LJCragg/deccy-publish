# Taxonomic Classification (Fungal ITS/18S)

**Tool:** EMU (Expectation-Maximisation Unique-read Resolution)
**Workflow:** Nextflow DSL2

EMU classifies each QC-filtered read against a curated reference database using expectation-maximisation, producing fractional abundance estimates per taxon.

## Reference Databases

| Database | Sequences | Description |
|----------|-----------|-------------|
| MIMt 18S+ITS (primary) | 61,924 | Curated fungal-focused sequences; NCBI taxonomy |
| UNITE ITS (alternative) | 100,176 | Broader coverage, general release 19.02.2025 |

## Classification Results

| Metric | Value |
|--------|-------|
| Total taxa detected | 1,306 |
| Fungal entries | 876 (67.1%) |
| Non-fungal | 429 (32.9%) |

**Per-sample fungal diversity:**

| Barcode | Fungal species | Fungal genera |
|---------|----------------|---------------|
| barcode19 | 135 | 107 |
| barcode20 | 282 | 206 |
| barcode21 | 375 | 244 |
| unclassified | 84 | 72 |

Reference databases and per-sample EMU outputs are not included in this repository (gitignored).
