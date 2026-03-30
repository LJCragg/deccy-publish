# Output Provenance

Summary of inputs used to generate each output table.

| Output | Source data | Notes |
|--------|-------------|-------|
| `tbl16s_01_qc_summary.csv` | Per-sample QC stats from preprocessing | All 6 samples pass consistency checks |
| `tbl16s_02_alpha_diversity.csv` | OTU97 threepoint-filtered abundance table | Shannon, Simpson, observed richness |
| `tbl16s_03_braycurtis_*.csv` | OTU97 threepoint-filtered abundance table | CLR-transformed before distance calculation |
| `tbl16s_04_comprehensive*.xlsx` | OTU97 table from reference-based clustering | 353 OTUs pre-filter; 66 degrader OTUs |
| `tbl16s_04_assoc97.csv` | Degrader whitelist matched against OTU97 table | 55 genera queried; 18 detected after threepoint |
| `tbl16s_04_degrader_inventory.csv` | Degrader whitelist × OTU97 match results | Evidence tier annotation (DIOXIN_CORE, BOTH, POP_BROAD) |
| `tbl16s_04_emu.csv` | EMU species-level abundance table (MIMt reference) | 437 species detected |
| `tbl16s_04_otu97_*.csv` | OTU97 tables before and after prevalence filter | Filter: ≥10 reads in all 3 replicates of ≥1 cohort |
| `tbl16s_05_taxa_function.csv` | Cross-reference of 16S degrader genera and targeted gene results | Integration of 16S and functional gene datasets |

**Canonical OTU tables** are in `../oven/`:
- `slashing97/threepoint/` — OTU97 prevalence-filtered (reference-based pipeline)
- `associated97/` — Degrader-associated subset
- `slashing/threepoint/` — OTU95 prevalence-filtered
