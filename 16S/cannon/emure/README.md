# Reference-Based OTU Clustering (16S)

EMU taxonomic classification against MIMt (31,426 sequences) followed by VSEARCH 97% identity clustering of classified reference sequences. This approach assigns taxonomy before clustering, ensuring confident classification for every OTU, but is reference-dependent — taxa absent from MIMt are not recovered.

**Reference database:** MIMt 16S (31,426 full-length sequences)
**Clustering identity:** 97% (species-level)
**Pre-clustering taxa:** 436 detected species
**OTU output:** 353 clusters (299 singletons, 84.7%)

The processed OTU tables are in [`../../oven/slashing97/`](../../oven/slashing97/) (prevalence-filtered) and [`../../oven/associated97/`](../../oven/associated97/) (degrader-associated subset). Formatted thesis tables are in [`../../outputs/tables/`](../../outputs/tables/).

See [`../../discussion/denovo/`](../../discussion/denovo/) for the de novo (reference-independent) alternative.
