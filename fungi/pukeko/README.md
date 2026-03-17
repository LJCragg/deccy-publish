# Pukeko — Fungal-Only ITS OTU Analysis

This directory contains a **refined fungal diversity analysis** derived from the parrot workflow, with all non-fungal taxa (plants, animals, oomycetes) removed to improve alpha and beta diversity signal.

## Rationale

The original [parrot](../parrot) workflow processed Morris ONT ITS data through EMU classification, but retained significant non-fungal contamination:
- **Plants (Streptophyta):** 97 entries in filtered data
- **Animals (Arthropoda, Chordata, Mollusca):** 138 entries
- **Green algae (Chlorophyta):** 14 entries
- **Oomycetes:** 13 entries (water molds, not true fungi)

This contamination inflated OTU counts and obscured true fungal diversity patterns. **Pukeko** filters to **true fungi only** for cleaner ecological analysis.

## Workflow

### Input Data
**Source:** [mapping/out/emu_slash/emu_abundance.tsv](../mapping/out/emu_slash/emu_abundance.tsv)  
- Pre-filtered EMU output (566 taxa)
- Includes plants, animals, algae, and fungi

### Step 1: Fungal-Only Filtering
```bash
awk 'BEGIN {FS="\t"; OFS="\t"} \
  NR==1 {print; next} \
  $9 ~ /^(Ascomycota|Basidiomycota|Chytridiomycota|Mucoromycota|Zoopagomycota|Blastocladiomycota|Olpidiomycota|Microsporidia|Cryptomycota)$/ \
  {print}' \
  mapping/out/emu_slash/emu_abundance.tsv > pukeko/emu_abundance_fungal.tsv
```

**Retained phyla (241 taxa):**
- Ascomycota: 80
- Chytridiomycota: 47
- Basidiomycota: 41
- Mucoromycota: 40
- Zoopagomycota: 23
- Microsporidia: 4
- Cryptomycota: 3
- Olpidiomycota: 2
- Blastocladiomycota: 1

**Excluded:**
- Oomycota (water molds, phylogenetically distinct)
- All plants, animals, algae, protists

### Step 2: Build Collapser Table
```bash
python pukeko/build_collapser_from_emu.py \
  --emu-combined pukeko/emu_abundance_fungal.tsv \
  --out pukeko/rout/collapser/collapser_abundance.tsv \
  --min-reads 10
```

**Output:** 241 taxa with ≥10 estimated reads

### Step 3: OTU Clustering (95% Identity)
**Taxa collapsed:** 120 unique taxids (with representative sequences)  
**OTU clusters:** 83 (60% singletons)

```bash
# Extract representative sequences
seqkit grep -n -f pukeko/rout/collapser/otu95/rep_seq_ids.txt \
  mapping/refdbs/mergemimt/mimt_clean.filtered.fasta \
  -o pukeko/rout/collapser/otu95/rep_seqs.fasta

# Cluster at 95% identity
vsearch --cluster_fast pukeko/rout/collapser/otu95/rep_seqs.fasta \
  --id 0.95 \
  --centroids pukeko/rout/collapser/otu95/centroids_0.95.fasta \
  --uc pukeko/rout/collapser/otu95/clusters_0.95.uc

# Collapse abundances by cluster
python pukeko/collapse_by_cluster.py \
  pukeko/rout/collapser/collapser_abundance.tsv \
  pukeko/rout/collapser/otu95/taxid2cluster.tsv \
  pukeko/rout/collapser/otu95/collapser_abundance_0.95.tsv \
  "ncbi tax number" \
  "Kingdom,Phylum,Class,Order,Family,Genus,Species" \
  "read_count"
```

**Output:** [rout/collapser/otu95/collapser_abundance_0.95.tsv](rout/collapser/otu95/collapser_abundance_0.95.tsv)  
- 182 rows (83 OTUs across 4 samples)
- Format matches parrot OTU table

### Step 4: Diversity Metrics
```bash
cd pukeko/components/metrics
python diversity_analysis.py \
  --input ../../rout/collapser/otu95/collapser_abundance_0.95.tsv \
  --outdir outputs \
  --transform clr \
  --value-col read_count
```

## Results Comparison

### Alpha Diversity

| Sample | **Fungal-Only (Pukeko)** | Original (Parrot) |
|--------|--------------------------|-------------------|
| | **Obs** | **Shannon** | **Simpson** | **Obs** | **Shannon** | **Simpson** |
| barcode19 | **37** | **3.04** | **0.917** | 185 | 3.97 | 0.958 |
| barcode20 | **49** | **3.09** | **0.908** | 272 | 3.81 | 0.946 |
| barcode21 | **70** | **3.51** | **0.950** | 334 | 3.90 | 0.951 |
| unclassified | **26** | **2.84** | **0.909** | 115 | 3.59 | 0.932 |

**Observations:**
- **79-80% reduction in observed OTUs** (contamination removed)
- **Shannon diversity decreased** (expected when removing non-fungal taxa)
- **Simpson evenness maintained** (0.91-0.95), indicating robust fungal community structure
- barcode21 still shows highest fungal richness (70 OTUs)

### Beta Diversity (Bray-Curtis Dissimilarity)

**Pukeko (Fungal-Only):**
|  | bc19 | bc20 | bc21 | uncl |
|--|------|------|------|------|
| **bc19** | 0.00 | 0.54 | **0.67** | 0.46 |
| **bc20** | 0.54 | 0.00 | **0.42** | 0.44 |
| **bc21** | **0.67** | **0.42** | 0.00 | 0.52 |
| **uncl** | 0.46 | 0.44 | 0.52 | 0.00 |

**Parrot (All Taxa):**
|  | bc19 | bc20 | bc21 | uncl |
|--|------|------|------|------|
| **bc19** | 0.00 | 0.40 | 0.44 | 0.36 |
| **bc20** | 0.40 | 0.00 | 0.30 | 0.35 |
| **bc21** | 0.44 | 0.30 | 0.00 | 0.32 |
| **uncl** | 0.36 | 0.35 | 0.32 | 0.00 |

**Observations:**
- **Increased dissimilarity** (0.42-0.67 vs 0.30-0.44) reveals stronger fungal community differentiation
- barcode19-barcode21 most dissimilar (0.67) — distinct fungal assemblages
- barcode20-barcode21 most similar (0.42) — shared fungal community structure
- **Clearer ecological signal** without plant/animal noise

## Key Insights

### Improved Signal
1. **Reduced noise:** Removed 318 non-fungal taxa from original 566
2. **Better resolution:** Fungal-specific OTUs show **stronger beta diversity gradients** (0.42-0.67 vs 0.30-0.44)
3. **Ecological clarity:** barcode21 emerges as fungal diversity hotspot (70 OTUs)

### Normalization Impact
- **Auto-normalized depth:** 1,569 reads/sample (95% of minimum)
- **Original depth range:** 1,652-7,409 reads
- **Lower than parrot** (17,826 reads) due to fungal-only filtering

### Remaining Taxa (83 OTUs)
**Dominant fungal groups:**
- **Ascomycota:** Diverse (80 species → ~35 OTUs)
- **Chytridiomycota:** Prevalent (47 species → ~20 OTUs)
- **Basidiomycota:** Moderate (41 species → ~15 OTUs)
- **Mucoromycota:** Glomeromycetes (arbuscular mycorrhizae)
- **Zoopagomycota:** Kickxellales (animal parasites)

## Directory Structure
```
pukeko/
├── emu_abundance_fungal.tsv          # Filtered EMU output (241 fungal taxa)
├── build_collapser_from_emu.py       # Custom script using estimated_counts
├── collapse_by_cluster.py            # OTU aggregation script
├── rout/
│   └── collapser/
│       ├── collapser_abundance.tsv   # Pre-OTU table (241 taxa, min-reads ≥10)
│       └── otu95/
│           ├── collapser_abundance_0.95.tsv  # Final OTU table (83 OTUs)
│           ├── centroids_0.95.fasta          # Representative sequences
│           ├── clusters_0.95.uc              # Vsearch clustering output
│           ├── taxid2cluster.tsv             # Taxid→OTU mapping
│           └── seq2cluster.tsv               # Sequence→OTU mapping
└── components/
    └── metrics/
        ├── diversity_analysis.py     # Diversity metrics script (from parrot)
        └── outputs/
            ├── alpha_diversity.csv
            ├── beta_diversity_braycurtis.csv
            ├── pcoa_coordinates.csv
            ├── otu_table_raw.csv
            ├── otu_table_normalized.csv
            ├── otu_table_clr.csv
            ├── taxonomy.csv
            ├── alpha_diversity_plots.png
            └── pcoa_ordination.png
```

## Files

### Outputs
- [rout/collapser/otu95/collapser_abundance_0.95.tsv](rout/collapser/otu95/collapser_abundance_0.95.tsv) — Primary OTU abundance matrix
- [components/metrics/outputs/alpha_diversity.csv](components/metrics/outputs/alpha_diversity.csv) — Shannon, Simpson, Chao1, observed OTUs
- [components/metrics/outputs/beta_diversity_braycurtis.csv](components/metrics/outputs/beta_diversity_braycurtis.csv) — Pairwise distance matrix
- [components/metrics/outputs/pcoa_coordinates.csv](components/metrics/outputs/pcoa_coordinates.csv) — Ordination coordinates
- [components/metrics/outputs/otu_table_clr.csv](components/metrics/outputs/otu_table_clr.csv) — CLR-transformed abundances

### Visualizations
- [components/metrics/outputs/alpha_diversity_plots.png](components/metrics/outputs/alpha_diversity_plots.png)
- [components/metrics/outputs/pcoa_ordination.png](components/metrics/outputs/pcoa_ordination.png)

## Next Steps

1. **Functional annotation:** Assign ecological roles (decomposer, pathogen, mycorrhizal, saprotroph)
2. **Taxonomic breakdown:** Analyze distribution of Ascomycota orders (Helotiales, Xylariales, etc.)
3. **Rarefaction curves:** Assess sampling completeness per sample
4. **Network analysis:** Co-occurrence patterns across fungal OTUs
5. **Comparative analysis:** Contrast fungal vs full dataset patterns

## Provenance
- **Created:** 2026-01-06
- **Parent workflow:** [parrot](../parrot)
- **Source data:** [mapping/out/emu_slash/emu_abundance.tsv](../mapping/out/emu_slash/emu_abundance.tsv) (566 taxa)
- **Filtered to:** 241 fungal taxa → 83 OTUs @ 95% similarity
- **Samples:** 4 (barcode19, barcode20, barcode21, unclassified)
- **Total fungal reads:** 10,740 (normalized to 1,569/sample)
- **Environment:** funcall conda env (vsearch 2.30.2, seqkit, Python 3)

## References
- Original workflow: [parrot/README.md](../parrot/README.md)
- EMU classification: [mapping/out/emu_slash/](../mapping/out/emu_slash/)
- Reference database: [mapping/refdbs/mergemimt/](../mapping/refdbs/mergemimt/)
- Diversity methods: Aitchison (1986) CLR transformation, Gower (1966) PCoA
