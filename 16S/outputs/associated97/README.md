# Associated OTU97 — Degrader-Linked Clusters

## What this is

OTU97 clusters from the **de novo VSEARCH clustering pipeline** (97% identity)
that match genera on the a priori POP/dioxin degrader whitelist. This replaces
the earlier EMU-based OTU95 fallback that only captured 2 degrader genera.

Generator: `claude/sixteen/prep/step0_generate_associated_denovo.py`

## Source data

| Component | Path | Description |
|-----------|------|-------------|
| OTU table (wide) | `clustering/06_tables/otu_table_combined.tsv` | 857 OTUs, 285 354 QC reads |
| OTU table (long) | `clustering/06_tables/otu_table_long.tsv` | One row per OTU × sample |
| Degrader whitelist | `sixteen/refdbs/dioxin_pop_degrader_genera.tsv` | 73 entries → 55 unique genera |

## Degrader genus whitelist

The whitelist (`dioxin_pop_degrader_genera.tsv`) contains genera with published
evidence for POP/dioxin degradation or co-metabolism. Three evidence tiers:

- **DIOXIN_CORE** — Direct dioxin/dibenzofuran degraders (e.g. Sphingomonas, Sphingobium)
- **BOTH** — Evidence for both dioxin and broader POP degradation (e.g. Dehalococcoides, Desulfomonile)
- **POP_BROAD** — Broader POP degradation including PCBs, PAHs, chlorophenols (e.g. Novosphingobium, Variovorax)

Sub-genus labels (e.g. `Sphingomonas_yanoikuyae`) are collapsed to genus level
for matching. After deduplication: **55 unique genera** queried.

## Filtering methodology

1. **Genus matching**: Each OTU's `Genus` column is compared against the 55-genus
   whitelist. Case-sensitive, exact genus match.

2. **Threepoint prevalence filter** (applied to the filtered version only):
   An OTU passes if it has read_count > 0 in **all 3 replicates** of at least one
   soil sample (SS1 = barcode13+14+15, SS2 = barcode16+17+18).

## Output files

| File | Rows | Description |
|------|------|-------------|
| `associated_OTU97.tsv` | 66 | Threepoint-filtered degrader OTUs |
| `associated_OTU97_unfiltered.tsv` | 70 | All degrader OTUs (no threepoint) |
| `associated_OTU97_long.tsv` | 364 | Long format (1 row per OTU × sample), threepoint-filtered |
| `associated_OTU97_summary.tsv` | 70 | One row per OTU with threepoint status flag |

### Column structure (wide files)

```
OTU_ID  total_reads  barcode13-18  method  identity  Kingdom  Phylum  Class  Order  Family  Genus  Species  matched_genera  evidence_category  threepoint_status
```

- `matched_genera` — the whitelist genus that this OTU matched
- `evidence_category` — DIOXIN_CORE / BOTH / POP_BROAD
- `threepoint_status` — pass / fail (only in summary and unfiltered; filtered file contains only pass)

## Results summary

### Detected genera (18 after threepoint, 19 before)

| Genus | OTUs (filtered) | Evidence tier |
|-------|-----------------|---------------|
| Sphingomonas | 25 | DIOXIN_CORE |
| Novosphingobium | 10 | POP_BROAD |
| Desulfitobacterium | 5 | BOTH |
| Dehalobacter | 4 (5 unfiltered) | BOTH |
| Sphingopyxis | 3 | DIOXIN_CORE |
| Sphingobium | 3 | DIOXIN_CORE |
| Sphingorhabdus | 2 | POP_BROAD |
| Ideonella | 2 | POP_BROAD |
| Desulfomonile | 2 | BOTH |
| Anaeromyxobacter | 2 | POP_BROAD |
| Variovorax | 1 | POP_BROAD |
| Serratia | 1 | POP_BROAD |
| Paracoccus | 1 | POP_BROAD |
| Hydrogenophaga | 1 | POP_BROAD |
| Flavobacterium | 1 | POP_BROAD |
| Aeromonas | 1 | POP_BROAD |
| Acidovorax | 1 | POP_BROAD |
| Acetobacterium | 1 | POP_BROAD |
| Agrobacterium | 0 (1 unfiltered) | POP_BROAD |

### Evidence tier breakdown (threepoint-filtered)

- DIOXIN_CORE: 25 OTUs
- BOTH: 12 OTUs
- POP_BROAD: 29 OTUs

### Total reads

- Threepoint-filtered: 18 471 reads across 66 OTUs
- Unfiltered: 18 508 reads across 70 OTUs
- Dominant OTU: OTU_10 (Sphingomonas, 8 811 reads = 47.7% of all degrader reads)

### Threepoint filter impact

4 OTUs removed by threepoint: all low-abundance (5–16 reads total), with zeros
in one or more replicates. The filter removes < 0.2% of total degrader reads.

### Notable absences

~36 genera from the whitelist were NOT detected in the de novo OTU97 table.
Key absences include Pseudomonas (0 OTUs despite being common in soil — likely
filtered at QC or chimera stage), Rhodococcus, Dehalococcoides, Burkholderia,
Arthrobacter, Mycobacterium. These may reflect genuine absence from the anonymized site
soil samples, primer bias (16S V1–V9 with specific primers), or low sequencing depth.

## Legacy

The previous `associated_OTU97.tsv` was actually OTU95 data from the EMU fallback
pipeline (only 2 degrader clusters at OTU97 in EMU). The legacy generator is
archived at `prep/step0_generate_otu97_associated_LEGACY_emu.py`.
