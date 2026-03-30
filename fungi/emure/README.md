# Reference-Based OTU Clustering (Fungal ITS/18S)

EMU taxonomic classification followed by phylum-level prefiltering and VSEARCH 95% identity clustering of classified reference sequences.

**Last updated:** 2026-03-10

---

## Method

1. **Prefilter:** Retain rows where phylum is a target fungal group (Ascomycota, Basidiomycota, Chytridiomycota, Mucoromycota, Zoopagomycota, Blastocladiomycota, Olpidiomycota, Microsporidia, Cryptomycota, Oomycota, Chlorophyta, Bacillariophyta). Removes plants and animals; retains fungi, borderline algae, and water molds.
2. **Collapser:** Converts prefiltered EMU table to VSEARCH-compatible format.
3. **Clustering:** VSEARCH at 95% identity using MIMt 18S+ITS reference sequences.
4. **Prevalence filter:** OTUs retained if ≥10 reads in all 3 barcodes.
5. **Target filtering:** Separate outputs for degrader-associated OTUs at genus, family, and phylum resolution.

---

## Run Statistics (2026-03-10)

| Step | Count |
|------|-------|
| EMU rows pre-filter | 1,305 |
| EMU rows post-filter | 913 (70.0%) |
| Estimated reads retained | 189,727 / 359,734 (52.7%) |
| OTU95 clusters | 308 |
| OTUs after prevalence filter | 91 (29.5%) |
| Reads retained after filter | 167,042 / 180,208 (92.7%) |
| Degrader-associated OTUs (genus-level) | 3 |

---

## Outputs

| File | Description |
|------|-------------|
| `00_prefilter/emu_abundance_optionB.tsv` | Prefiltered EMU abundance table |
| `00_prefilter/emu_filter_summary_optionB.tsv` | Prefilter summary |
| `01_collapser/collapser_input.tsv` | Collapser-format input for VSEARCH |
| `02_reference_seqs/rep_seqs.fasta` | Representative sequences for detected taxa |
| `03_cluster/centroids_0.95.fasta` | VSEARCH cluster centroids |
| `03_cluster/clusters_0.95.uc` | VSEARCH cluster assignments |
| `04_tables/otu_table_combined.tsv` | OTU table (wide format, all barcodes) |
| `04_tables/otu_table_long.tsv` | OTU table (long format) |
| `threepoint/emure_associated_threepoint.tsv` | Degrader-associated OTUs (genus-level) |
| `threepoint/emure_associated_family_threepoint.tsv` | Degrader-associated OTUs (family-level) |
| `threepoint/emure_associated_phylum_threepoint.tsv` | Degrader-associated OTUs (phylum-level) |
| `threepoint/emure_otu95_threepoint.tsv` | All OTUs passing prevalence filter |
