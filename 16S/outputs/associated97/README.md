# Degrader-Associated OTU Clusters

OTU97 clusters from the de novo VSEARCH clustering pipeline (97% identity) that match genera on the a priori POP/dioxin-degrader whitelist.

---

## Degrader Whitelist

55 unique genera with published evidence for POP/dioxin degradation or co-metabolism. Three evidence tiers:

| Tier | Description | Example genera |
|------|-------------|----------------|
| DIOXIN_CORE | Direct dioxin/dibenzofuran degraders | Sphingomonas, Sphingobium, Sphingopyxis |
| BOTH | Evidence for dioxin and broader POP degradation | Dehalococcoides, Desulfomonile, Desulfitobacterium |
| POP_BROAD | Broader POP degradation (PCBs, PAHs, chlorophenols) | Novosphingobium, Variovorax, Ideonella |

---

## Filtering

1. Genus-level match against 55-genus whitelist (exact, case-sensitive).
2. Prevalence filter: OTU present with >0 reads in all 3 replicates of at least one cohort (SS1 = BC13–15; SS2 = BC16–18).

---

## Results (threepoint-filtered)

| Genus | OTUs | Evidence tier |
|-------|------|---------------|
| Sphingomonas | 25 | DIOXIN_CORE |
| Novosphingobium | 10 | POP_BROAD |
| Desulfitobacterium | 5 | BOTH |
| Dehalobacter | 4 | BOTH |
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

**Total:** 66 OTUs across 18 genera (18,471 reads). Dominant OTU: *Sphingomonas* OTU_10 (8,811 reads, 47.7% of degrader reads).

Evidence tier breakdown: DIOXIN_CORE 25 OTUs, BOTH 12 OTUs, POP_BROAD 29 OTUs.

4 OTUs removed by prevalence filter (all low-abundance, <0.2% of degrader reads).

---

## Output Files

| File | Rows | Description |
|------|------|-------------|
| `associated_OTU.tsv` | 66 | Threepoint-filtered degrader OTUs (wide format) |
| `associated_OTU_summary.tsv` | 70 | All degrader OTUs with threepoint status flag |
| `long_associated.tsv` | 364 | Long format — one row per OTU × sample |

**Column notes:** `matched_genera` — whitelist genus matched; `evidence_category` — DIOXIN_CORE / BOTH / POP_BROAD; `threepoint_status` — pass / fail.
