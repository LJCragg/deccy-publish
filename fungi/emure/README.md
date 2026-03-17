# EMURE (EMU-derived OTU95 workflow)

Last updated: March 10, 2026

## Purpose

`emure/` builds 95% OTUs from the Morris EMU abundance table, then applies:

1. threepoint reproducibility filtering
2. associated-target genus filtering

This workflow now applies a taxonomy-only prefilter before clustering.

## Primary Input

- EMU abundance table: `../mapping/morris/out/emu_morris/emu_abundance.tsv`
- Reference mapping: `../mapping/refdbs/mergemimt/mimt_mapping.fixed.tsv`
- Reference FASTA: `../mapping/refdbs/mergemimt/mimt_clean.filtered.fasta`

## Current Prefilter Rule (Option B)

Implemented in `scripts/filter_emu_taxa.py` and called by `scripts/run_emure.sh`.

No abundance cutoff is used. Rows are retained only if `phylum` is one of:

- `Ascomycota`
- `Basidiomycota`
- `Chytridiomycota`
- `Mucoromycota`
- `Zoopagomycota`
- `Blastocladiomycota`
- `Olpidiomycota`
- `Microsporidia`
- `Cryptomycota`
- `Oomycota`
- `Chlorophyta`
- `Bacillariophyta`

This keeps fungi plus borderline algae and water molds, while removing clear non-target groups (for example plants and animals).

## Pipeline Steps

Run:

```bash
bash scripts/run_emure.sh
```

`run_emure.sh` executes:

1. `filter_emu_taxa.py` -> `00_prefilter/emu_abundance_optionB.tsv`
2. `build_its_collapser.py` -> `01_collapser/collapser_input.tsv`
3. `collapser_otu95.sh` (VSEARCH at `id=0.95`)
4. `build_otu_tables.py` -> wide and long OTU tables
5. `filter_threepoint.py` (`>=10` reads in all three barcodes)
6. `filter_associated.py` (target genera whitelist — 56 genera)
7. `filter_associated_phylum.py` (target phylum whitelist — broad-resolution)
8. `filter_associated_family.py` (target family whitelist — mid-resolution, 19 families)

## Key Outputs

- Prefiltered EMU table: `00_prefilter/emu_abundance_optionB.tsv`
- Prefilter summary: `00_prefilter/emu_filter_summary_optionB.tsv`
- OTU table (wide): `04_tables/otu_table_combined.tsv`
- OTU table (long): `04_tables/otu_table_long.tsv`
- Threepoint OTUs: `threepoint/emure_otu95_threepoint.tsv`
- Threepoint long: `threepoint/emure_otu95_threepoint_long.tsv`
- Threepoint summary: `threepoint/emure_otu95_threepoint_summary.tsv`
- Associated OTUs: `threepoint/emure_associated_threepoint.tsv`
- Non-associated OTUs: `threepoint/emure_non_associated_threepoint.tsv`
- Associated OTUs (phylum): `threepoint/emure_associated_phylum_threepoint.tsv`
- Non-associated OTUs (phylum): `threepoint/emure_non_associated_phylum_threepoint.tsv`
- Associated OTUs (family): `threepoint/emure_associated_family_threepoint.tsv`
- Non-associated OTUs (family): `threepoint/emure_non_associated_family_threepoint.tsv`
- Run log: `emure.log`

## Current Run Snapshot (March 10, 2026)

From the Option-B rerun:

- Prefilter retained `913 / 1305` rows (`69.962%`)
- Prefilter retained `189,726.86 / 359,734.00` estimated reads (`52.741%`)
- OTUs at 95%: `308`
- Threepoint passing OTUs: `91` (`29.5%` of 308)
- Threepoint reads retained: `167,042 / 180,208` (`92.7%`)
- Associated passing OTUs: `3`
- Non-associated passing OTUs: `88`
- Matched associated genus in this run: `Acaulospora`

Phylum filter snapshot (pending re-run):

- Associated phylum OTUs: pending
- Matched phyla: `Basidiomycota`, `Ascomycota`, `Mucoromycota`

Reference files:

- `00_prefilter/emu_filter_summary_optionB.tsv`
- `04_tables/otu_table_combined.tsv`
- `threepoint/emure_otu95_threepoint.tsv`
- `threepoint/emure_associated_threepoint.tsv`

## Notes

- `emure.log` is appended to by clustering steps and may contain multiple runs.
- All clustering is performed at 95% identity (`OTU_ID=0.95` in `run_emure.sh`).
- This workflow does not apply prevalence or minimum-read filtering before OTU clustering.
