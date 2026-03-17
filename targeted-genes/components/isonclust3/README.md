# isONclust3 component

This component clusters long reads using [isONclust3](https://github.com/aljpetri/isONclust3) to produce per-cluster FASTQs and an assignment table (`final_clusters.tsv`). It is designed to consume Greedhunt outputs and can operate in two modes:

- per-target: clusters each `<product>/<barcode>.fastq` produced by Greedhunt, yielding per-target, per-barcode clusters
- per-barcode: clusters merged per-barcode FASTQs if Greedhunt was run with `output_mode=merged` or `both`

## Inputs and outputs
- Inputs:
  - per-target: `results/greedhunt/<product>/*.fastq`
  - per-barcode: `results/greedhunt/merged/*.fastq`
- Outputs:
  - `results/isonclust3/<product>/<label>/final_clusters.tsv` (per-target)
  - `results/isonclust3/<product>/<label>/fastq_files/` (directory of per-cluster FASTQs)
  - `results/isonclust3/barcode/<label>/final_clusters.tsv` (per-barcode)

## Latest run + rerun tips
- Full Porch → Greedhunt → isONclust3 flow validated with `conda run -n roadtrip snakemake --cores 8 --latency-wait 60 isonclust3_all`.
- Greedhunt per-target FASTQs now include the `unclassified` labels, so per-target clustering picks up the reads Postage flagged as rare.
- To reprocess a specific target/barcode pair, target the desired TSV directly, e.g. `snakemake --cores 8 results/isonclust3/clca/barcode01/final_clusters.tsv`. Logs land in `results/isonclust3/logs/<product>_<label>.log` (or `<label>.log` for per-barcode mode).

## Configuration (`config/components/isonclust3.yml`)
```yaml
isonclust3:
  input_mode: per-target         # or per-barcode
  greedhunt_dir: results/greedhunt
  merged_subdir: merged
  outdir: results/isonclust3
  threads: 8
  mode: ont
  extra: "--post-cluster"      # forwarded to isONclust3
```

Enable the component by setting in `config/roadtrip.yml`:
```yaml
components:
  isonclust3: true
```

## Environment and installation
isONclust3 is a Rust binary. Ensure Rust (cargo) is available in the `roadtrip` environment, then install the tool once:

```bash
# optional: inside the roadtrip conda env
cargo install isONclust3
# or build from source
# git clone https://github.com/aljpetri/isONclust3.git && cd isONclust3 && cargo build --release && export PATH="$PWD/target/release:$PATH"
```

You can add `rust` to `envs/roadtrip.yml` to provision cargo in the environment. The Snakemake rules assume `isONclust3` is on PATH; set one up using either installation method above.

## Entry rule
Run a dry-run to inspect the DAG and expected outputs:

```bash
snakemake -n isonclust3_all
```

Then execute with cores:

```bash
snakemake --cores 8 isonclust3_all
```
