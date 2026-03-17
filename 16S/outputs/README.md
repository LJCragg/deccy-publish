# 16S Outputs Directory

This directory contains all generated data and visualizations for 16S rRNA analysis.

## Folder Structure

```
outputs/
├── data/               # CSV data files for figures
│   ├── core/           # Core analyses (phylum, alpha, beta diversity, pipeline)
│   ├── taxa/           # Taxa-focused visualizations (genus bars, pies, heatmaps)
│   ├── qc/             # QC metrics (read lengths, filtering stats)
│   └── associated/     # Degrader-associated taxa analyses
│
├── plots/              # Generated figure images
│   ├── png/            # PNG format (150 dpi preview, 300 dpi final)
│   │   ├── core/
│   │   ├── taxa/
│   │   └── comparison/
│   └── pdf/            # PDF format (publication-ready vectors)
│       ├── core/
│       ├── taxa/
│       └── comparison/
│
├── tables/             # Summary tables for thesis/publication
├── associated97/       # Degrader-associated OTU cluster data
└── README_audit_trail.md  # Audit log of all figure generation
```

## Figure Naming Convention

- `fig16s_XX_description.csv/png/pdf`
  - XX: Figure number (01-18+)
  - Suffixes indicate variants (a/b for top5/top10)

## Key Figure Categories

### Core (01-07)
- 01: Phylum composition
- 03: Alpha diversity metrics
- 04: PCoA/beta diversity
- 07: Pipeline filtering funnel

### Taxa (08-18)
- 08-10: Pie charts (overall, per-barcode, per-cohort)
- 11: Read length bar chart
- 12-13: Bar charts (per-sample, cohort comparison)
- 14: Genus abundance heatmaps
- 18: **Clean visualizations (no "Other" category)**

### Associated/Degrader (05-06, 15-17)
- 05: Degrader heatmap
- 15: Full vs Associated comparison
- 16: Degrader-highlighted bars
- 17: Lollipop plot

## Regenerating Figures

```bash
cd claude/sixteen/
python fig16s_18_clean_top_genera.py   # Clean top genera (no "Other")
python plot_all_figures.py              # All original figures
```
