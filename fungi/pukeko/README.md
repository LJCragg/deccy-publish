# Fungi-Only OTU Analysis

Refined fungal diversity analysis with non-fungal eukaryotes removed prior to OTU clustering.

**Created:** 2026-01-06

---

## Rationale

The full EMU abundance table includes plants, animals, algae, and water molds alongside fungi. This directory retains only true fungi (Ascomycota, Basidiomycota, Chytridiomycota, Mucoromycota, Zoopagomycota, Blastocladiomycota, Olpidiomycota, Microsporidia, Cryptomycota), then re-clusters at 95% identity.

---

## Filtering Results

**Input:** 566 taxa (full EMU output)
**Retained:** 241 fungal taxa

| Phylum | Taxa retained |
|--------|--------------|
| Ascomycota | 80 |
| Chytridiomycota | 47 |
| Basidiomycota | 41 |
| Mucoromycota | 40 |
| Zoopagomycota | 23 |
| Microsporidia | 4 |
| Cryptomycota | 3 |
| Olpidiomycota | 2 |
| Blastocladiomycota | 1 |

---

## OTU Clustering Results

**Clustering:** VSEARCH 95% identity
**Input:** 120 unique taxids with ≥10 estimated reads
**Output:** 83 OTU clusters (60% singletons)

---

## Alpha Diversity

| Sample | Observed OTUs | Shannon | Simpson |
|--------|--------------|---------|---------|
| barcode19 | 37 | 3.04 | 0.917 |
| barcode20 | 49 | 3.09 | 0.908 |
| barcode21 | 70 | 3.51 | 0.950 |
| unclassified | 26 | 2.84 | 0.909 |

barcode21 has the highest fungal OTU richness (70 OTUs). CLR transformation applied; data normalised to 1,569 reads per sample (95% of minimum).

---

## Beta Diversity (Bray-Curtis)

|  | bc19 | bc20 | bc21 |
|--|------|------|------|
| bc19 | 0.00 | 0.54 | 0.67 |
| bc20 | 0.54 | 0.00 | 0.42 |
| bc21 | 0.67 | 0.42 | 0.00 |

---

## Output Files

| File | Description |
|------|-------------|
| `emu_abundance_fungal.tsv` | Filtered EMU abundance table (241 fungal taxa) |
| `rout/collapser/otu95/collapser_abundance_0.95.tsv` | OTU95 abundance table (83 OTUs, wide format) |
| `components/metrics/outputs/alpha_diversity.csv` | Alpha diversity indices (Shannon, Simpson, observed OTUs) |
| `components/metrics/outputs/beta_diversity_braycurtis.csv` | Pairwise Bray-Curtis distance matrix |
| `components/metrics/outputs/pcoa_coordinates.csv` | PCoA ordination coordinates |
| `components/metrics/outputs/otu_table_clr.csv` | CLR-transformed OTU abundance table |
| `components/metrics/outputs/alpha_diversity_plots.png` | Alpha diversity figure |
| `components/metrics/outputs/pcoa_ordination.png` | PCoA ordination figure |
