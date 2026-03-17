# Audit Trail: 16S Results Visualisation

Each entry records the exact inputs, transformations, and outputs.


## Step0-Associated
- **Timestamp**: 2026-02-21 14:26:49
- **Script**: `step0_generate_otu97_associated.py`
- **Inputs**:
  - `/home/uca/chover/sixteen/oven/slashing/threepoint/collapser_abundance_0.95_threepoint.tsv`
  - `/home/uca/chover/sixteen/refdbs/dioxin_pop_degrader_genera.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/associated97/associated_OTU97.tsv`
  - `/home/uca/chover/claude/sixteen/outputs/associated97/associated_OTU97_summary.tsv`
- **Notes**: Resolution: OTU95. 23 clusters from 14 genera. Genera: Acetobacterium, Acidovorax, Agrobacterium, Anaeromyxobacter, Dehalobacter, Desulfitobacterium, Flavobacterium, Hydrogenophaga, Ideonella, Novosphingobium, Sphingobium, Sphingomonas, Sphingopyxis, Sphingorhabdus. FALLBACK from OTU97 (only 2 clusters).


## Tbl16S-01
- **Timestamp**: 2026-02-21 14:26:49
- **Script**: `tbl16s_01_qc_pipeline.py`
- **Inputs**:
  - `/home/uca/chover/writeup/tables/sixteen_prefilter_qc_summary.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/tables/tbl16s_01_qc_summary.csv`
- **Notes**: Per-sample QC summary. All 6 samples pass consistency checks.


## Fig16S-07
- **Timestamp**: 2026-02-21 14:26:49
- **Script**: `fig16s_07_pipeline_funnel.py`
- **Inputs**:
  - `/home/uca/chover/writeup/tables/sixteen_prefilter_qc_summary.tsv`
  - `pipeline documentation`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_07_funnel.csv`
- **Notes**: Funnel table. Some values are placeholders pending step0 output.


## Fig16S-01
- **Timestamp**: 2026-02-21 14:26:49
- **Script**: `fig16s_01_phylum_bars.py`
- **Inputs**:
  - `/home/uca/chover/sixteen/components/output/phylum_per_sample_data.csv`
  - `/home/uca/chover/sixteen/components/output/phylum_cohort_summary.csv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_01_per_sample.csv`
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_01_cohort_means.csv`
- **Notes**: Phylum stacked bar data. Per-sample sums verified ~1.0.


## Fig16S-02
- **Timestamp**: 2026-02-21 14:26:50
- **Script**: `fig16s_02_genus_top15.py`
- **Inputs**:
  - `/home/uca/chover/sixteen/rout/emu/emu_abundance.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_02_genus_top15.csv`
- **Notes**: Top 15 genera grouped bar data. #1 = Fonticella (9.6%).


## Tbl16S-02
- **Timestamp**: 2026-02-21 14:26:50
- **Script**: `tbl16s_02_alpha_diversity.py`
- **Inputs**:
  - `/home/uca/chover/writeup/assist/sixteen_otu97_threepoint_alpha_diversity.tsv`
  - `/home/uca/chover/writeup/assist/sixteen_otu97_threepoint_cohort_summary.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/tables/tbl16s_02_alpha_diversity.csv`
- **Notes**: Alpha diversity per sample + cohort summaries. Cross-checked against cohort file.


## Fig16S-03
- **Timestamp**: 2026-02-21 14:26:50
- **Script**: `fig16s_03_alpha_diversity.py`
- **Inputs**:
  - `/home/uca/chover/writeup/assist/sixteen_otu97_threepoint_alpha_diversity.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_03_alpha_long.csv`
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_03_alpha_summary.csv`
- **Notes**: Alpha diversity long format for box/strip plots.


## Tbl16S-03
- **Timestamp**: 2026-02-21 14:26:50
- **Script**: `tbl16s_03_beta_diversity.py`
- **Inputs**:
  - `/home/uca/chover/writeup/assist/sixteen_otu97_threepoint_braycurtis.tsv`
  - `/home/uca/chover/writeup/assist/sixteen_otu97_threepoint_braycurtis_summary.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/tables/tbl16s_03_braycurtis_matrix.csv`
  - `/home/uca/chover/claude/sixteen/outputs/tables/tbl16s_03_braycurtis_summary.csv`
- **Notes**: Between-cohort mean = 0.721, within = 0.146. Draft value 0.577 does NOT match actual 0.721.


## Fig16S-04
- **Timestamp**: 2026-02-21 14:26:50
- **Script**: `fig16s_04_beta_pcoa.py`
- **Inputs**:
  - `/home/uca/chover/writeup/assist/sixteen_otu97_threepoint_braycurtis.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_04_pcoa_coords.csv`
- **Notes**: PCoA from Bray-Curtis. PC1=95.4%, PC2=2.9%. Between-cohort mean=0.721.


## Tbl16S-04
- **Timestamp**: 2026-02-21 14:26:50
- **Script**: `tbl16s_04_degrader_inventory.py`
- **Inputs**:
  - `/home/uca/chover/claude/sixteen/outputs/associated97/associated_OTU97.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/tables/tbl16s_04_degrader_inventory.csv`
- **Notes**: 23 OTU97 degrader clusters, 15 genera. 17 clusters present in all 6 samples.


## Fig16S-05
- **Timestamp**: 2026-02-21 14:26:50
- **Script**: `fig16s_05_degrader_heatmap.py`
- **Inputs**:
  - `/home/uca/chover/claude/sixteen/outputs/associated97/associated_OTU97.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_05_heatmap_wide.csv`
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_05_heatmap_long.csv`
- **Notes**: CLR heatmap data. 23 OTUs, pseudocount=0.5. Structural zeros: 11/138.


## Fig16S-06
- **Timestamp**: 2026-02-21 14:26:50
- **Script**: `fig16s_06_degrader_genus.py`
- **Inputs**:
  - `/home/uca/chover/claude/sixteen/outputs/associated97/associated_OTU97.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_06_degrader_genus.csv`
- **Notes**: Genus-level degrader summary. 14 genera, 2 multi-genus OTU(s) split proportionally.


## Tbl16S-05
- **Timestamp**: 2026-02-21 14:26:50
- **Script**: `tbl16s_05_taxa_function.py`
- **Inputs**:
  - `/home/uca/chover/claude/sixteen/outputs/associated97/associated_OTU97_summary.tsv`
  - `/home/uca/chover/sixteen/refdbs/dioxin_pop_degrader_genera.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/tables/tbl16s_05_taxa_function.csv`
- **Notes**: Taxa-to-function validation. 14/58 genera detected, 3 validated by targeted gene sequencing.


## Fig16S-06
- **Timestamp**: 2026-02-21 16:30:55
- **Script**: `fig16s_06_degrader_genus.py`
- **Inputs**:
  - `/home/uca/chover/claude/sixteen/outputs/associated97/associated_OTU97.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_06_degrader_genus.csv`
- **Notes**: Genus-level degrader summary. 15 genera. 2 multi-genus OTU(s) assigned to representative genus.


## Fig16S-08
- **Timestamp**: 2026-02-23 16:49:27
- **Script**: `fig16s_08_pie_top5_overall.py`
- **Inputs**:
  - `/home/uca/chover/sixteen/rout/emu/emu_abundance.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_08_pie_top5_overall.csv`
- **Notes**: Top 5 genera: Fonticella, Xylanivirga, Acetivibrio, Oxobacter, Thermoclostridium. #1 = Fonticella (9.6%).


## Fig16S-09
- **Timestamp**: 2026-02-23 16:49:27
- **Script**: `fig16s_09_pie_top5_per_barcode.py`
- **Inputs**:
  - `/home/uca/chover/sixteen/rout/emu/emu_abundance.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_09_pie_top5_per_barcode.csv`
- **Notes**: Small-multiples pie charts (6 panels). Consistent color mapping using overall top 10.


## Fig16S-10
- **Timestamp**: 2026-02-23 16:49:28
- **Script**: `fig16s_10_pie_top5_per_cohort.py`
- **Inputs**:
  - `/home/uca/chover/sixteen/rout/emu/emu_abundance.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_10_pie_top5_per_cohort.csv`
- **Notes**: Cohort-level pie charts (2 panels): SS1 vs SS2 top 5 genera.


## Fig16S-11
- **Timestamp**: 2026-02-23 16:49:28
- **Script**: `fig16s_11_bar_readlength.py`
- **Inputs**:
  - `/home/uca/chover/writeup/tables/sixteen_prefilter_qc_summary.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_11_bar_readlength.csv`
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_11_readlength_summary.csv`
- **Notes**: Mean read lengths before/after QC. Overall change: -3.1%.


## Fig16S-12
- **Timestamp**: 2026-02-23 16:49:40
- **Script**: `fig16s_12_bar_topN_per_barcode.py`
- **Inputs**:
  - `/home/uca/chover/sixteen/rout/emu/emu_abundance.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_12a_bar_top5_per_barcode.csv`
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_12b_bar_top10_per_barcode.csv`
- **Notes**: Stacked bar data for top 5 and top 10 genera per barcode.


## Fig16S-13
- **Timestamp**: 2026-02-23 16:49:41
- **Script**: `fig16s_13_bar_topN_per_cohort.py`
- **Inputs**:
  - `/home/uca/chover/sixteen/rout/emu/emu_abundance.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_13a_bar_top5_cohort.csv`
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_13b_bar_top10_cohort.csv`
- **Notes**: Cohort comparison (SS1 vs SS2) for top 5 and top 10 genera with t-test p-values.


## Fig16S-14
- **Timestamp**: 2026-02-23 16:49:41
- **Script**: `fig16s_14_heatmap_topN.py`
- **Inputs**:
  - `/home/uca/chover/sixteen/rout/emu/emu_abundance.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_14a_heatmap_top5_long.csv`
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_14b_heatmap_top10_long.csv`
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_14a_heatmap_top5_clr.csv`
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_14b_heatmap_top10_clr.csv`
- **Notes**: Heatmap data (raw pct + CLR) for top 5 and top 10 genera.


## Fig16S-15
- **Timestamp**: 2026-02-23 16:49:49
- **Script**: `fig16s_15_comparison_all_vs_associated.py`
- **Inputs**:
  - `/home/uca/chover/sixteen/rout/emu/emu_abundance.tsv`
  - `/home/uca/chover/claude/sixteen/outputs/associated97/associated_OTU97.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_15_comparison_all_vs_associated.csv`
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_15_enrichment.csv`
- **Notes**: Full vs degrader-associated comparison. 14 associated genera detected.


## Fig16S-16
- **Timestamp**: 2026-02-23 16:51:10
- **Script**: `fig16s_16_highlight_degraders.py`
- **Inputs**:
  - `/home/uca/chover/sixteen/rout/emu/emu_abundance.tsv`
  - `/home/uca/chover/sixteen/refdbs/dioxin_pop_degrader_genera.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_16_highlight_degraders_top10.csv`
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_16_highlight_degraders_top15.csv`
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_16_highlight_degraders_per_sample.csv`
- **Notes**: Bar charts with 58 degrader genera highlighted.


## Fig16S-17
- **Timestamp**: 2026-02-23 16:51:11
- **Script**: `fig16s_17_lollipop_associated.py`
- **Inputs**:
  - `/home/uca/chover/claude/sixteen/outputs/associated97/associated_OTU97.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_17_lollipop_associated.csv`
- **Notes**: Lollipop plot data for 14 degrader-associated genera.


## Tbl16S-04-Comprehensive
- **Timestamp**: 2026-02-24 11:49:07
- **Script**: `tbl16s_04_comprehensive.py`
- **Inputs**:
  - `/home/uca/chover/sixteen/cannon/emu/emu_abundance.tsv`
  - `/home/uca/chover/claude/sixteen/outputs/associated97/associated_OTU97.tsv`
  - `/home/uca/chover/sixteen/oven/slashing97/threepoint/collapser_abundance_0.97_threepoint.tsv`
  - `/home/uca/chover/sixteen/oven/slashing_trace97/threepoint/collapser_abundance_0.97_threepoint.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/tables/tbl16s_04_comprehensive.xlsx`
  - `/home/uca/chover/claude/sixteen/outputs/tables/tbl16s_04_emu.csv`
  - `/home/uca/chover/claude/sixteen/outputs/tables/tbl16s_04_assoc97.csv`
  - `/home/uca/chover/claude/sixteen/outputs/tables/tbl16s_04_otu97_after_3pt.csv`
  - `/home/uca/chover/claude/sixteen/outputs/tables/tbl16s_04_otu97_before_3pt.csv`
- **Notes**: Per-barcode top-20 genus tables across EMU, Associated OTU97, OTU97 after threepoint, OTU97 before threepoint (trace). Output: Excel workbook + 4 individual CSVs.


## Tbl16S-04-Reads
- **Timestamp**: 2026-02-24 13:10:32
- **Script**: `tbl16s_04_comprehensive_reads.py`
- **Inputs**:
  - `/home/uca/chover/sixteen/cannon/emu/emu_abundance.tsv`
  - `/home/uca/chover/claude/sixteen/outputs/associated97/associated_OTU97.tsv`
  - `/home/uca/chover/sixteen/oven/slashing97/threepoint/collapser_abundance_0.97_threepoint.tsv`
  - `/home/uca/chover/sixteen/oven/slashing_trace97/threepoint/collapser_abundance_0.97_threepoint.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/tables/tbl16s_04_comprehensive_reads.xlsx`
  - `/home/uca/chover/claude/sixteen/outputs/tables/tbl16s_04_reads_emu.csv`
  - `/home/uca/chover/claude/sixteen/outputs/tables/tbl16s_04_reads_assoc97.csv`
  - `/home/uca/chover/claude/sixteen/outputs/tables/tbl16s_04_reads_otu97_after_3pt.csv`
  - `/home/uca/chover/claude/sixteen/outputs/tables/tbl16s_04_reads_otu97_before_3pt.csv`
- **Notes**: Per-barcode top-20 genus tables using read counts (not rel. abundance). Ranked by total reads across all 6 barcodes. Same four sources as tbl16s_04_comprehensive.


## Fig16S-19
- **Timestamp**: 2026-03-01 11:11:51
- **Script**: `fig16s_19_associated_otu97_barchart.py`
- **Inputs**:
  - `/home/uca/chover/claude/sixteen/outputs/associated97/associated_OTU97.tsv`
  - `/home/uca/chover/claude/sixteen/denovo/06_tables/otu_table_long.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_19a_associated_otu97_top5_genus.png`
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_19b_associated_otu97_top10_genus.png`
- **Notes**: Top 5 and 10 associated OTU97 genera, Race vs Claude sixteen. Race top genus: Sphingomonas, Claude top genus: Sphingomonas.


## Step0-Associated-DeNovo
- **Timestamp**: 2026-03-01 11:33:38
- **Script**: `step0_generate_associated_denovo.py`
- **Inputs**:
  - `/home/uca/chover/claude/sixteen/denovo/06_tables/otu_table_combined.tsv`
  - `/home/uca/chover/claude/sixteen/denovo/06_tables/otu_table_long.tsv`
  - `/home/uca/chover/sixteen/refdbs/dioxin_pop_degrader_genera.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/associated97/associated_OTU97.tsv`
  - `/home/uca/chover/claude/sixteen/outputs/associated97/associated_OTU97_unfiltered.tsv`
  - `/home/uca/chover/claude/sixteen/outputs/associated97/associated_OTU97_long.tsv`
  - `/home/uca/chover/claude/sixteen/outputs/associated97/associated_OTU97_summary.tsv`
- **Notes**: De novo VSEARCH OTU97. Unfiltered: 70 OTUs, 19 genera. Threepoint: 66 OTUs, 18 genera (both=50, SS1=6, SS2=10). Total reads: 18,471 (filtered), 18,508 (all). Not detected: Achromobacter, Acinetobacter, Agromyces, Alcaligenes, Arthrobacter, Azoarcus, Bacillus, Beijerinckia, Bradyrhizobium, Burkholderia....


## Tbl16S-04
- **Timestamp**: 2026-03-01 11:53:29
- **Script**: `tbl16s_04_degrader_inventory.py`
- **Inputs**:
  - `/home/uca/chover/claude/sixteen/outputs/associated97/associated_OTU97.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/tables/tbl16s_04_degrader_inventory.csv`
- **Notes**: 66 OTU97 degrader clusters (de novo), 18 genera. 50 present in all 6 barcodes.


## Tbl16S-04-Comprehensive
- **Timestamp**: 2026-03-01 11:54:10
- **Script**: `tbl16s_04_comprehensive.py`
- **Inputs**:
  - `/home/uca/chover/sixteen/cannon/emu/emu_abundance.tsv`
  - `/home/uca/chover/claude/sixteen/outputs/associated97/associated_OTU97.tsv`
  - `/home/uca/chover/sixteen/oven/slashing97/threepoint/collapser_abundance_0.97_threepoint.tsv`
  - `/home/uca/chover/sixteen/oven/slashing_trace97/threepoint/collapser_abundance_0.97_threepoint.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/tables/tbl16s_04_comprehensive.xlsx`
  - `/home/uca/chover/claude/sixteen/outputs/tables/tbl16s_04_emu.csv`
  - `/home/uca/chover/claude/sixteen/outputs/tables/tbl16s_04_assoc97.csv`
  - `/home/uca/chover/claude/sixteen/outputs/tables/tbl16s_04_otu97_after_3pt.csv`
  - `/home/uca/chover/claude/sixteen/outputs/tables/tbl16s_04_otu97_before_3pt.csv`
- **Notes**: Per-barcode top-20 genus tables across EMU, Associated OTU97, OTU97 after threepoint, OTU97 before threepoint (trace). Output: Excel workbook + 4 individual CSVs.


## Tbl16S-04-Reads
- **Timestamp**: 2026-03-01 11:54:14
- **Script**: `tbl16s_04_comprehensive_reads.py`
- **Inputs**:
  - `/home/uca/chover/sixteen/cannon/emu/emu_abundance.tsv`
  - `/home/uca/chover/claude/sixteen/outputs/associated97/associated_OTU97.tsv`
  - `/home/uca/chover/sixteen/oven/slashing97/threepoint/collapser_abundance_0.97_threepoint.tsv`
  - `/home/uca/chover/sixteen/oven/slashing_trace97/threepoint/collapser_abundance_0.97_threepoint.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/tables/tbl16s_04_comprehensive_reads.xlsx`
  - `/home/uca/chover/claude/sixteen/outputs/tables/tbl16s_04_reads_emu.csv`
  - `/home/uca/chover/claude/sixteen/outputs/tables/tbl16s_04_reads_assoc97.csv`
  - `/home/uca/chover/claude/sixteen/outputs/tables/tbl16s_04_reads_otu97_after_3pt.csv`
  - `/home/uca/chover/claude/sixteen/outputs/tables/tbl16s_04_reads_otu97_before_3pt.csv`
- **Notes**: Per-barcode top-20 genus tables using read counts (not rel. abundance). Ranked by total reads across all 6 barcodes. Same four sources as tbl16s_04_comprehensive.


## Tbl16S-05
- **Timestamp**: 2026-03-01 11:54:19
- **Script**: `tbl16s_05_taxa_function.py`
- **Inputs**:
  - `/home/uca/chover/claude/sixteen/outputs/associated97/associated_OTU97_summary.tsv`
  - `/home/uca/chover/sixteen/refdbs/dioxin_pop_degrader_genera.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/tables/tbl16s_05_taxa_function.csv`
- **Notes**: Taxa-to-function validation. 19/58 genera detected, 3 validated by targeted gene sequencing.


## Fig16S-05
- **Timestamp**: 2026-03-01 11:54:27
- **Script**: `fig16s_05_degrader_heatmap.py`
- **Inputs**:
  - `/home/uca/chover/claude/sixteen/outputs/associated97/associated_OTU97.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_05_heatmap_wide.csv`
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_05_heatmap_long.csv`
- **Notes**: CLR heatmap data (de novo OTU97). 66 OTUs, pseudocount=0.5. Structural zeros: 32/396.


## Fig16S-06
- **Timestamp**: 2026-03-01 11:54:27
- **Script**: `fig16s_06_degrader_genus.py`
- **Inputs**:
  - `/home/uca/chover/claude/sixteen/outputs/associated97/associated_OTU97.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_06_degrader_genus.csv`
- **Notes**: Genus-level degrader summary (de novo OTU97). 18 genera.


## Fig16S-15
- **Timestamp**: 2026-03-01 11:54:28
- **Script**: `fig16s_15_comparison_all_vs_associated.py`
- **Inputs**:
  - `/home/uca/chover/sixteen/rout/emu/emu_abundance.tsv`
  - `/home/uca/chover/claude/sixteen/outputs/associated97/associated_OTU97.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_15_comparison_all_vs_associated.csv`
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_15_enrichment.csv`
- **Notes**: Full vs degrader-associated comparison (de novo OTU97). 18 associated genera detected.


## Fig16S-17
- **Timestamp**: 2026-03-01 11:54:29
- **Script**: `fig16s_17_lollipop_associated.py`
- **Inputs**:
  - `/home/uca/chover/claude/sixteen/outputs/associated97/associated_OTU97.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_17_lollipop_associated.csv`
- **Notes**: Lollipop plot data (de novo OTU97) for 18 degrader genera.


## Fig16S-19
- **Timestamp**: 2026-03-01 11:54:30
- **Script**: `fig16s_19_associated_otu97_barchart.py`
- **Inputs**:
  - `/home/uca/chover/claude/sixteen/outputs/associated97/associated_OTU97.tsv`
  - `/home/uca/chover/claude/sixteen/denovo/06_tables/otu_table_long.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_19a_associated_otu97_top5_genus.png`
  - `/home/uca/chover/claude/sixteen/outputs/figures/fig16s_19b_associated_otu97_top10_genus.png`
- **Notes**: Top 5 and 10 associated OTU97 genera — both de novo. Race top: Sphingomonas, Claude top: Sphingomonas.


## Threepoint-DeNovo
- **Timestamp**: 2026-03-01 12:32:07
- **Script**: `threepoint/filter_threepoint_denovo.py`
- **Inputs**:
  - `/home/uca/chover/claude/sixteen/denovo/06_tables/otu_table_combined.tsv`
  - `/home/uca/chover/claude/sixteen/denovo/06_tables/otu_table_long.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/threepoint/outputs/otu97_threepoint.tsv`
  - `/home/uca/chover/claude/sixteen/threepoint/outputs/otu97_threepoint_long.tsv`
  - `/home/uca/chover/claude/sixteen/threepoint/outputs/otu97_threepoint_summary.tsv`
- **Notes**: De novo VSEARCH OTU97 threepoint filter (min_reads >= 10). Input: 857 OTUs. Passing: 357 (both=120, SS1=101, SS2=136). Removed: 500. Reads retained: 227,834/242,557 (93.9%).


## Associated-Threepoint-DeNovo
- **Timestamp**: 2026-03-01 12:32:13
- **Script**: `threepoint/filter_associated_threepoint.py`
- **Inputs**:
  - `/home/uca/chover/claude/sixteen/threepoint/outputs/otu97_threepoint.tsv`
  - `/home/uca/chover/sixteen/refdbs/dioxin_pop_degrader_genera.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/threepoint/outputs/associated_threepoint.tsv`
  - `/home/uca/chover/claude/sixteen/threepoint/outputs/associated_threepoint_long.tsv`
  - `/home/uca/chover/claude/sixteen/threepoint/outputs/associated_threepoint_summary.tsv`
  - `/home/uca/chover/claude/sixteen/threepoint/outputs/non_associated_threepoint.tsv`
- **Notes**: Degrader genus matching on threepoint-filtered OTU97. Input: 357 threepoint OTUs. Associated: 32 OTUs, 14 genera (17,387 reads). Non-associated: 325 OTUs (210,447 reads). Tiers: DIOXIN_CORE=12, BOTH=5, POP_BROAD=15.


## Threepoint-Emure
- **Timestamp**: 2026-03-06 15:29:41
- **Script**: `threepoint/filter_threepoint_emure.py`
- **Inputs**:
  - `/home/uca/chover/claude/sixteen/emure/04_tables/otu_table_combined.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/threepoint/outputs/emure/emure_otu97_threepoint.tsv`
  - `/home/uca/chover/claude/sixteen/threepoint/outputs/emure/emure_otu97_threepoint_long.tsv`
  - `/home/uca/chover/claude/sixteen/threepoint/outputs/emure/emure_otu97_threepoint_summary.tsv`
- **Notes**: EMU reference-based OTU97 threepoint filter (min_reads >= 10.0). Input: 353 OTUs. Passing: 245 (both=125, SS1=30, SS2=90). Removed: 108. Reads retained: 276,635.0/279,969.0 (98.8%).


## Threepoint-Emure
- **Timestamp**: 2026-03-06 17:28:18
- **Script**: `threepoint/filter_threepoint_emure.py`
- **Inputs**:
  - `/home/uca/chover/claude/sixteen/emure/04_tables/otu_table_combined.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/threepoint/outputs/emure/emure_otu97_threepoint.tsv`
  - `/home/uca/chover/claude/sixteen/threepoint/outputs/emure/emure_otu97_threepoint_long.tsv`
  - `/home/uca/chover/claude/sixteen/threepoint/outputs/emure/emure_otu97_threepoint_summary.tsv`
- **Notes**: EMU reference-based OTU97 threepoint filter (min_reads >= 10.0). Input: 353 OTUs. Passing: 245 (both=125, SS1=30, SS2=90). Removed: 108. Reads retained: 276,629.0/279,964.0 (98.8%).


## Threepoint-DeNovo
- **Timestamp**: 2026-03-14 16:05:35
- **Script**: `threepoint/filter_threepoint_denovo.py`
- **Inputs**:
  - `/home/uca/chover/claude/sixteen/denovo/06_tables/otu_table_combined.tsv`
  - `/home/uca/chover/claude/sixteen/denovo/06_tables/otu_table_long.tsv`
- **Outputs**:
  - `/home/uca/chover/claude/sixteen/threepoint/outputs/otu97_threepoint.tsv`
  - `/home/uca/chover/claude/sixteen/threepoint/outputs/otu97_threepoint_long.tsv`
  - `/home/uca/chover/claude/sixteen/threepoint/outputs/otu97_threepoint_summary.tsv`
- **Notes**: De novo VSEARCH OTU97 threepoint filter (min_reads >= 10). Input: 857 OTUs. Passing: 357 (both=120, SS1=101, SS2=136). Removed: 500. Reads retained: 227,834/242,557 (93.9%).

