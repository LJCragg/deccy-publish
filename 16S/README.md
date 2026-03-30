# 16S rRNA Amplicon Analysis

**Bacterial community profiling from full-length 16S rRNA amplicons (Oxford Nanopore Technologies)**

Taxonomic classification and OTU-level clustering of bacterial communities, with targeted identification of dioxin/POP-degrader associated taxa.

**Last updated:** 2026-02-20

---

## Samples

| Barcode | Cohort | Description |
|---------|--------|-------------|
| BC13 | SS1 | Biological replicate 1 |
| BC14 | SS1 | Biological replicate 2 |
| BC15 | SS1 | Biological replicate 3 |
| BC16 | SS2 | Biological replicate 1 |
| BC17 | SS2 | Biological replicate 2 |
| BC18 | SS2 | Biological replicate 3 |

Reads were subsampled to 120,000 per barcode for depth normalisation. Post-QC retention: approximately 47,700 reads per barcode (Q-filter + length 500–2000 bp + 15 bp end-crop).

---

## Key Results

| Metric | Value |
|--------|-------|
| EMU taxa detected (species-level) | 437 |
| OTU95 clusters (pre-filter) | 258 |
| OTU97 clusters (pre-filter) | 353 |
| OTU95 after prevalence filter | 192 |
| Degrader-associated OTUs | 18 (14 target genera) |
| Dominant phyla | Bacillota, Pseudomonadota, Bacteroidota (>99% combined) |
| SS1 Shannon diversity (OTU97) | 2.56 ± 0.06 |
| SS2 Shannon diversity (OTU97) | 2.79 ± 0.17 |

**Phylum composition:**
- Bacillota: 72.9 ± 6.8% (SS1), 64.6 ± 2.8% (SS2)
- Pseudomonadota: 24.1 ± 1.3% (SS1), 23.5 ± 0.2% (SS2)
- Bacteroidota: 1.1 ± 0.1% (SS1), 8.4 ± 1.1% (SS2) — 7.6-fold depletion in SS1

**Degrader genera detected (top four by OTU count):** Sphingomonas (8 OTUs), Novosphingobium (4 OTUs), Desulfitobacterium (2 OTUs), Acetobacterium (1 OTU).

**Sequencing:** ONT MinION R10.4.1, SUP basecalling (Dorado). Full-length 16S V1–V9 amplicons (~1500 bp). Taxonomic classification: EMU against MIMt database (31,426 sequences). OTU clustering: VSEARCH at 95% and 97% identity.

**Prevalence filter:** OTUs retained if present with ≥10 reads in all 3 replicates of at least one cohort.

---

## Directory Structure

| Directory | Contents |
|-----------|----------|
| `outputs/tables/` | Formatted thesis tables (QC summary, alpha diversity, Bray-Curtis, degrader inventory, OTU97 pre/post-filter) |
| `oven/` | Processed OTU tables at 95% and 97% identity — pre- and post-filter, wide and long format; degrader-associated subsets; OTU comparison figures |
| `cannon/emure/` | Reference-based OTU clustering documentation |
| `discussion/denovo/` | De novo OTU clustering (alternative approach; comparison with reference-based results) |
| `components/basic/` | Sphingomonadaceae genus-level summaries (TSV) |
| `components/output/` | Preliminary phylum-level figures and summary tables |

Thesis figures and statistical analysis tables are in [`../r_analysis/`](../r_analysis/).

---

## Output Tables (`outputs/tables/`)

| File | Description |
|------|-------------|
| `tbl16s_01_qc_summary.csv` | Per-sample read counts and QC retention |
| `tbl16s_02_alpha_diversity.csv` | Alpha diversity indices (Shannon, Simpson) per sample |
| `tbl16s_03_braycurtis_matrix.csv` | Bray-Curtis dissimilarity matrix |
| `tbl16s_03_braycurtis_summary.csv` | Pairwise Bray-Curtis summary |
| `tbl16s_04_comprehensive.xlsx` | Comprehensive OTU abundance table (all OTU97, wide format) |
| `tbl16s_04_comprehensive_reads.xlsx` | As above, raw read counts |
| `tbl16s_04_assoc97.csv` | Degrader-associated OTU97 abundances |
| `tbl16s_04_degrader_inventory.csv` | Degrader genus detection inventory |
| `tbl16s_04_emu.csv` | EMU species-level abundance table |
| `tbl16s_04_otu97_after_3pt.csv` | OTU97 table post-prevalence filter |
| `tbl16s_04_otu97_before_3pt.csv` | OTU97 table pre-filter |
| `tbl16s_05_taxa_function.csv` | Taxa-to-function linkage table |
