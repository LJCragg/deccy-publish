# De Novo OTU Clustering (16S — Discussion)

**Alternative 97% OTU clustering pipeline using VSEARCH de novo clustering of raw reads.**

This approach clusters QC-filtered reads directly at 97% identity, then assigns taxonomy post-hoc using dual methods (SINTAX k-mer bootstrap + VSEARCH global alignment against MIMt). It is reference-independent: taxa absent from MIMt are still recovered and counted.

**Last updated:** 2026-02-26

---

## Comparison with Reference-Based Pipeline

| Aspect | Reference-based | De novo |
|--------|----------------|---------|
| Input to clustering | MIMt reference sequences for detected taxa | QC-filtered amplicon reads |
| OTU97 count | 353 (84.7% singletons) | 857 (0% singletons) |
| Taxonomy method | EMU (pre-clustering) | SINTAX + VSEARCH global (post-clustering) |
| Novel taxa | Not recovered | Recovered |
| Centroid sequences | Database entries | Actual sample reads |

Both pipelines use the same QC parameters and sample structure (6 barcodes, two cohorts; ~47,700 reads per barcode after filtering).

---

## Outputs

Figures comparing the two pipelines are in [`../../../r_analysis/figures/sixteen/discussion_denovo_emubased/`](../../../r_analysis/figures/sixteen/discussion_denovo_emubased/). Statistical tables are in [`../../../r_analysis/outputs/sixteen/discussion_denovo_emubased/`](../../../r_analysis/outputs/sixteen/discussion_denovo_emubased/).
