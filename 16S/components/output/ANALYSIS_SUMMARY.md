# EMU Baseline Taxonomic Diversity Analysis - Summary

**Analysis Date:** January 26, 2026  
**Dataset:** EMU abundance data (pre-filtering baseline)  
**Samples:** 6 barcodes (3 fuzzsample, 3 drysample cohorts)

---

## Key Findings

### 1. Taxonomic Richness
- **Genus richness range:** 127-204 genera per sample
- **Species richness range:** 208-322 species per sample
- High taxonomic diversity across all samples
- No unmapped/unclassified entries included in analysis

### 2. Prominent Taxa
**Top Genus:** *Fonticella* (0.5767 total abundance)  
**Top Species:** *Fonticella tunisiensis* (0.5767 total abundance)

This indicates strong dominance of *Fonticella* in the sampled community, accounting for approximately 57.67% of total abundance across samples.

### 3. Alpha Diversity Metrics

#### Genus Level
- **Shannon diversity range:** 3.276 - 4.273
- Moderate to high diversity across samples

#### Species Level  
- **Shannon diversity range:** 3.549 - 4.642
- Higher diversity at species level compared to genus (expected)

#### Statistical Comparisons (Mann-Whitney U Tests)
- **No significant differences** (p < 0.05) between fuzzsample and drysample cohorts for:
  - Shannon diversity (genus and species levels)
  - Simpson diversity (genus and species levels)
  - Observed richness (genus and species levels)

**Interpretation:** Despite different environmental conditions, the two cohorts show statistically similar alpha diversity patterns.

### 4. Beta Diversity Analysis

#### Genus Level PCoA
- **PC1 variance explained:** 82.4%
- **PC2 variance explained:** 9.6%
- Strong separation along first principal component

#### Species Level PCoA
- **PC1 variance explained:** 81.9%
- **PC2 variance explained:** 9.9%
- Similar pattern to genus-level analysis

#### PERMANOVA Results
See Excel workbook for detailed statistical test results comparing community composition between cohorts.

---

## Output Files

### Excel Workbook: `emu_baseline_results.xlsx`
**11 sheets containing:**
1. **Summary** - Overview metrics and sample counts
2. **Taxonomic_Richness** - Per-sample genus/species richness
3. **Alpha_Diversity** - Shannon, Simpson, observed richness per sample
4. **Alpha_Statistics** - Mann-Whitney U test results
5. **PERMANOVA** - Beta diversity statistical tests
6. **BrayCurtis_Genus** - Genus-level distance matrix
7. **BrayCurtis_Species** - Species-level distance matrix
8. **PCoA_Genus** - Genus-level ordination coordinates
9. **PCoA_Species** - Species-level ordination coordinates
10. **Top_Genera** - Top 20 genera by abundance
11. **Top_Species** - Top 20 species by abundance

### Figures (Publication-Ready, 300 DPI)
1. **fig1_taxonomic_richness.png** - Genus/species richness boxplots by cohort
2. **fig2_alpha_diversity.png** - 6-panel alpha diversity comparison (Shannon, Simpson, richness × genus/species)
3. **fig3_beta_diversity_pcoa.png** - PCoA ordination plots for genus and species levels
4. **fig4_prominent_taxa.png** - Top 15 genera and species bar charts

---

## Next Steps for Results Section

### Immediate Comparisons
1. **Post-OTU Collapsing Analysis**
   - Run same analysis on `sixteen/cannon/collapsed95/` data
   - Expected outcome: Reduced taxonomic richness, consolidated diversity metrics
   - Compare richness, alpha/beta diversity, and prominent taxa

2. **Post-Threepoint Filtering Analysis**
   - Run analysis on `sixteen/oven/slashing/threepoint/` data
   - Expected outcome: Further reduction to reproducible/prevalent OTUs
   - Assess impact on diversity metrics and community structure

### Masters-Level Results Section Structure

```
Results Section: Taxonomic Diversity Analysis

1. Baseline Community Composition (EMU)
   - Taxonomic richness across samples
   - Dominant taxa identification
   - Alpha diversity patterns
   - Beta diversity and sample clustering
   
2. Impact of OTU Clustering (95% identity)
   - Changes in taxonomic richness
   - Effect on diversity metrics
   - Retention of prominent taxa
   
3. Effect of Threepoint Filtering
   - Reduction to core/reproducible taxa
   - Diversity metric stability
   - Focus on ecologically relevant taxa
   
4. Comparative Analysis
   - Side-by-side diversity comparisons
   - Statistical significance of filtering steps
   - Biological interpretation of patterns
```

---

## Technical Notes

### Cohort Mapping
- **fuzzsample:** barcode13, barcode14, barcode15
- **drysample:** barcode16, barcode17, barcode18

### Methods Summary
- **Alpha diversity:** Shannon index, Simpson index, observed richness (scikit-bio)
- **Beta diversity:** Bray-Curtis dissimilarity with PCoA ordination
- **Statistical tests:**
  - Mann-Whitney U test for alpha diversity comparisons (scipy.stats)
  - PERMANOVA for beta diversity comparisons (scikit-bio, 999 permutations)
- **Significance threshold:** p < 0.05

### Software Environment
- **Conda environment:** sixteen
- **Python packages:** pandas, numpy, matplotlib, seaborn, scipy, scikit-bio, openpyxl

---

## Data Quality Notes

1. **Unmapped/unclassified entries removed** - Analysis focuses on taxonomically assigned reads
2. **Abundance normalization** - Relative abundances sum to ~1.0 per sample
3. **Taxonomic completeness** - All samples have genus and species-level assignments
4. **Sample size** - n=3 per cohort (limited statistical power, interpret with caution)

---

## Citation Recommendations

For methods section, cite:
- **scikit-bio:** Caporaso JG, et al. (2010) QIIME allows analysis of high-throughput community sequencing data. Nature Methods 7:335-336
- **Shannon diversity:** Shannon CE (1948) A mathematical theory of communication. Bell System Technical Journal 27:379-423
- **Bray-Curtis:** Bray JR, Curtis JT (1957) An ordination of the upland forest communities of southern Wisconsin. Ecological Monographs 27:325-349
- **PERMANOVA:** Anderson MJ (2001) A new method for non-parametric multivariate analysis of variance. Austral Ecology 26:32-46
