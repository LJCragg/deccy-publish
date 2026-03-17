# EMU Baseline Analysis Components

This directory contains comprehensive taxonomic diversity analysis scripts for EMU abundance data, designed for masters-level bioinformatics results sections.

## Scripts

### `emu_baseline_analysis.py`

Performs baseline (pre-filtering) taxonomic diversity analysis including:

- **Taxonomic Richness**: Genus and species counts per sample and cohort
- **Prominent Taxa**: Top 20 taxa by abundance and prevalence
- **Alpha Diversity**: Shannon, Simpson, and observed richness indices
- **Beta Diversity**: Bray-Curtis dissimilarity with PCoA ordination
- **Statistical Tests**: Mann-Whitney U (alpha diversity), PERMANOVA (beta diversity)
- **Outputs**: Single Excel workbook with 11 sheets + 4 publication-ready figures

## Requirements

Install required Python packages in the `sixteen` conda environment:

```bash
conda activate sixteen
conda install -c conda-forge pandas numpy matplotlib seaborn scipy scikit-bio openpyxl
```

Or using pip:

```bash
pip install pandas numpy matplotlib seaborn scipy scikit-bio openpyxl
```

## Usage

```bash
conda activate sixteen

python sixteen/components/emu_baseline_analysis.py \
    --input sixteen/cannon/emu/emu_abundance.tsv \
    --output sixteen/components/output/emu_baseline_results.xlsx \
    --figures sixteen/components/output \
    --top-taxa 20
```

## Outputs

### Excel Workbook Sheets:
1. **Summary** - Overview metrics
2. **Taxonomic_Richness** - Per-sample richness
3. **Alpha_Diversity** - Diversity indices per sample
4. **Alpha_Statistics** - Mann-Whitney U test results
5. **PERMANOVA** - Beta diversity statistical tests
6. **BrayCurtis_Genus** - Genus-level distance matrix
7. **BrayCurtis_Species** - Species-level distance matrix
8. **PCoA_Genus** - Genus-level ordination coordinates
9. **PCoA_Species** - Species-level ordination coordinates
10. **Top_Genera** - Top 20 genera by abundance
11. **Top_Species** - Top 20 species by abundance

### Figures:
1. `fig1_taxonomic_richness.png` - Genus/species richness boxplots
2. `fig2_alpha_diversity.png` - Alpha diversity metrics by cohort
3. `fig3_beta_diversity_pcoa.png` - PCoA ordination plots
4. `fig4_prominent_taxa.png` - Top 15 genera and species bar charts

## Cohort Mapping

- **fuzzsample**: barcode13, barcode14, barcode15
- **drysample**: barcode16, barcode17, barcode18

## Future Comparisons

This baseline analysis will be compared against:
1. Post-OTU collapsing results (95% identity clustering)
2. Post-threepoint prominence filtering results
