# MAFFT component

This component aligns per-target consensus sequences with MAFFT. It supports two modes: per-barcode target alignment (default) or per-gene grouping using minimap2 to assign each consensus sequence to a target gene. The current setup uses gene-based grouping to reduce cross-gene gaps and a fragment split to keep short sequences from distorting the full-length alignment.

## Inputs and outputs
- Inputs:
  - `results/medaka/<target>/<barcode>/merged_consensus.fasta`
  - `admin/targets.fa` (required for gene grouping and reference inclusion)
  - optional `extra_fasta` (filtered by gene suffix)
- Outputs (per target/gene):
  - `results/mafft/<target>/input.fasta`
  - `results/mafft/<target>/alignment.fasta`
  - `results/mafft/<target>/fragments.fasta`
  - `results/mafft/<target>/fragments.alignment.fasta`
  - `results/mafft/<target>/alignment.with_fragments.fasta`
  - `results/mafft/logs/<target>.log`

## Configuration (`config/components/mafft.yml`)
```yaml
mafft:
  medaka_dir: results/medaka
  ref: admin/targets.fa
  outdir: results/mafft
  group_by: gene          # "barcode" or "gene"
  min_length: 650         # sequences shorter than this go to fragments
  include_targets: true
  extra_fasta: ""
  add_fragments: true     # if true, add fragments into main alignment output
  minimap_preset: asm5
  min_identity: 0.0
  threads: 4
```

Notes:
- Gene grouping uses the last underscore-delimited token in a target header as the gene suffix (e.g. `foo_bar_clca` -> `clca`). Targets and extra sequences must follow that convention.
- When `group_by: gene`, unassigned consensus sequences (no minimap2 hit or below `min_identity`) are excluded from all gene alignments.

## Reasoning
- Early alignments had large gaps, mostly driven by mixed genes and short fragments.
- The response was to:
  - Assign each consensus to a specific gene using minimap2 best hit.
  - Split sequences below `min_length` into a fragment set, aligning them separately.
  - Optionally add fragments back to the main alignment with `--addfragments`.

## Status (successes and failures)
Successes:
- Gene-based grouping works; per-gene alignments are produced and reference targets are included.
- Fragment splitting at `min_length=650` separated short sequences (clca: 1 fragment; catB2: 3 fragments), which keeps main alignments cleaner than the combined set.

Failures/limits:
- Gap rates remain high for `clca` and `catB2`; adding fragments back (`alignment.with_fragments.fasta`) increases overall gap fraction.
- `min_length=450` did not split any sequences, indicating most problematic records are longer than expected.
- `ntDAa` and `BpHc` have too few sequences to evaluate alignment quality.
- MAFFT is currently invoked without explicit `--thread`, so runtime is effectively single-threaded even if Snakemake reserves more threads.

## Environment and installation
The component requires `mafft` and (for gene grouping) `minimap2` on PATH in the `roadtrip` environment. The rules reference `envs/roadtrip.yml`.

## Entry rule
```bash
snakemake -n mafft_all
snakemake --cores 4 mafft_all
```

Enable the component in `config/roadtrip.yml`:
```yaml
components:
  mafft: true
```
