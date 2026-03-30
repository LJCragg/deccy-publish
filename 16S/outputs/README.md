# 16S Output Tables

Formatted summary tables for the 16S rRNA analysis, generated from the processed OTU data in `../oven/`.

## Tables (`tables/`)

| File | Description |
|------|-------------|
| `tbl16s_01_qc_summary.csv` | Per-sample read counts and QC retention statistics |
| `tbl16s_02_alpha_diversity.csv` | Alpha diversity indices (Shannon, Simpson, observed richness) per sample |
| `tbl16s_03_braycurtis_matrix.csv` | Bray-Curtis dissimilarity matrix (6 × 6) |
| `tbl16s_03_braycurtis_summary.csv` | Pairwise Bray-Curtis distances with cohort labels |
| `tbl16s_04_comprehensive.xlsx` | OTU97 abundance table — all passing OTUs, wide format (relative abundance) |
| `tbl16s_04_comprehensive_reads.xlsx` | OTU97 abundance table — raw read counts |
| `tbl16s_04_assoc97.csv` | Degrader-associated OTU97 abundances (relative) |
| `tbl16s_04_reads_assoc97.csv` | Degrader-associated OTU97 abundances (read counts) |
| `tbl16s_04_degrader_inventory.csv` | Degrader genus detection inventory with evidence tier |
| `tbl16s_04_emu.csv` | EMU species-level abundance table |
| `tbl16s_04_reads_emu.csv` | EMU species-level raw read counts |
| `tbl16s_04_otu97_after_3pt.csv` | OTU97 table post-prevalence filter |
| `tbl16s_04_reads_otu97_after_3pt.csv` | OTU97 raw counts post-filter |
| `tbl16s_04_otu97_before_3pt.csv` | OTU97 table pre-filter |
| `tbl16s_04_reads_otu97_before_3pt.csv` | OTU97 raw counts pre-filter |
| `tbl16s_05_taxa_function.csv` | Taxa-to-function linkage: 16S genera to targeted gene detections |

## Associated OTUs (`associated97/`)

Degrader-associated OTU clusters — see `associated97/README.md`.
