# R Analysis — Figures and Tables

Statistical analysis and figure generation for the 16S bacterial and fungal ITS/18S amplicon datasets. All figures are 300 dpi PNG. Tables are TSV.

---

## Figures (`figures/`)

### 16S Bacterial Analysis (`figures/sixteen/`)

| Subdirectory | Contents |
|---|---|
| `qc/` | Read length distribution |
| `threepoint/` | Taxonomic composition (genus, phylum, order, family), OTU richness, alpha diversity, beta diversity PCoA, OTU97 filtering transition |
| `associated/` | Dioxin-degrader genus bar charts, CLR heatmaps, degrader detection summary |
| `network/` | Co-occurrence network plots (genus, family, order levels); degrader subnetwork |
| `discussion_denovo_emubased/` | Comparison of reference-based and de novo OTU clustering approaches |

### Fungal ITS/18S Analysis (`figures/funcall/`)

| Subdirectory | Contents |
|---|---|
| `threepoint/` | OTU95 composition (phylum, genus), richness, top OTUs, filtering retention, contamination profiles, phylum panels |
| `associated/` | Degrader-associated OTU heatmap; plant symbiont detections |

---

## Tables (`outputs/`)

### 16S Bacterial Analysis (`outputs/sixteen/`)

| Subdirectory | Key tables |
|---|---|
| `qc/` | Per-sample and overall QC summaries; read length statistics |
| `emu/` | EMU species-level alpha diversity, Bray-Curtis distances, phylum composition, sample summaries |
| `threepoint/` | OTU97 alpha diversity (pre- and post-filter); Bray-Curtis distances; OTU filtering transition summaries |
| `associated/` | Dioxin-degrader OTU evidence summary |
| `network/` | Co-occurrence network nodes, edges, and modules at genus, OTU97, family, and order levels |

### Fungal ITS/18S Analysis (`outputs/funcall/`)

| Subdirectory | Key tables |
|---|---|
| `threepoint/` | OTU95 summaries (overall, phylum); alpha diversity (pre- and post-filter); Bray-Curtis distances; top OTU lists; contamination assessment; EMU phylum counts |
| `associated/` | Degrader-associated OTU inventory |

---

## Software

R (ggplot2, vegan, igraph). Analyses use CLR-transformed abundances for compositional data. Beta diversity computed as Bray-Curtis dissimilarity.
