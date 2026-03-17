# Funcall fungal (18S) mapping workflow

Goal: mirror the “sixteen” 16S pipeline for ONT fungal/18S reads using an EMU + MIMt-18S database, then optionally cluster at 95% identity.

## Monorepo path note

This pipeline now lives under `/home/uca/chover/funcall`. If you copy absolute paths below, replace `/home/uca/funcall` with `/home/uca/chover/funcall`.

## Planned flow
1) Input: basecalled/demuxed FASTQs (flattened under `house/output/noclass/fastqpass/` or similar).  
2) Classify: Nextflow Emu pipeline (18S DB) → per-barcode outputs + combined `emu_abundance.tsv` (target: `rout_fungi/emu/`).  
3) Collapser build: lift abundances to taxids with read-count thresholds (analysis vs trace) → `collapser_abundance.tsv` in `rout_fungi/collapser/` and `rout_fungi/collapser_trace/`.  
4) OTU95 (vsearch): comparative + per-sample clustering → cluster tables + membership (“composition checker”).  
5) Sandbox: mirror under `cannon_fungi/` for parameter sweeps.

## Environment
- Recommended env (mirrors 16S “sixteen” stack): `sixtran/funcall.yml` (name: `funcall`) with Nextflow 25, Java 17, EMU, minimap2/samtools, seqkit, vsearch, etc.  
  - Create: `mamba env create -n funcall -f sixtran/funcall.yml` (or `mamba env update -n funcall -f sixtran/funcall.yml`).  
  - Activate: `mamba activate funcall`; sanity: `nextflow -version`, `emu --version`.
- A lighter prototype env remains at `env/funcall.yml` (Python 3.12 + EMU/vsearch); prefer `sixtran/funcall.yml` for Nextflow runs.

## Reference DBs
- **MIMt 18S** (legacy): `mapping/refdbs/mimt18s/` with helper scripts; outputs `emu_db/` + `mimt18s_mapping.fixed.tsv`.
- **MIMt 18S + ITS (combined, current)**: `mapping/refdbs/mergemimt/`
  - Inputs: `MIMt-18S_M2c_25_10_tax.fna.gz`, `MIMt-ITS_fun_M2c_25_10_tax.fna.gz`, and NCBI taxdump under `ncbisrc/taxdump/`.
  - Normalised FASTA + mapping: `mimt_clean.filtered.fasta` (61,924 seqs), `mimt_seq2tax.tsv` (seq→taxid, no header).
  - EMU DB: `mapping/refdbs/mergemimt/emu_db` built with  
    `mamba run -n funcall emu build-database --sequences mimt_clean.filtered.fasta --seq2tax mimt_seq2tax.tsv --ncbi-taxonomy refdbs/mimt/ncbi_tax emu_db`.
- **UNITE ITS (general release 19.02.2025)**: `mapping/refdbs/unit/`
  - Source: `sh_general_release_dynamic_19.02.2025.fasta` (from `sh_general_release_19.02.2025.tgz`).
  - Normalised FASTA + mapping: `unite_clean.filtered.fasta` (100,176 seqs), `unite_seq2tax.tsv` (seq→taxid, no header).
  - Taxdump: symlinked `refdbs/unite/ncbi_tax -> ../mimt18s/src/taxdump` (NCBI dump reused).
  - EMU DB: `mapping/refdbs/unit/emu_db` built with  
    `mamba run -n funcall emu build-database --sequences unite_clean.filtered.fasta --seq2tax unite_seq2tax.tsv --ncbi-taxonomy refdbs/unite/ncbi_tax emu_db`.

## Run the fungal Emu Nextflow (full scale)
From the repo root:
```bash
mamba activate funcall
nextflow run sixtran/main.nf \
  --db /home/uca/funcall/mapping/refdbs/mergemimt/emu_db \
  --reads "/home/uca/funcall/oven/slashing/filtered/*.fastq" \
  --outdir /home/uca/funcall/mapping/out/emu_slash \
  --cpus 8 \
  -ansi-log false
```
Outputs land under `mapping/out/emu_full/` (per-sample folders + combined `emu_abundance.tsv`). Use `-resume` on re-runs.

## Latest run (porechop+chopper filtered)
- Command:
  ```bash
  mamba activate funcall
  nextflow run sixtran/main.nf \
    --db /home/uca/funcall/mapping/refdbs/mergemimt/emu_db \
    --reads "/home/uca/funcall/oven/slashing/filtered/*.fastq" \
    --outdir /home/uca/funcall/mapping/out/emu_slash \
    --cpus 8 \
    -ansi-log false
  ```
- Inputs: `oven/slashing/filtered/*.fastq` (HEADCROP=15, TAILCROP=15, MINQ=10, MINLEN=300, MAXLEN=3500 via slashing pipeline).
- Outputs: `mapping/out/emu_slash/` per-sample tables + combined `emu_abundance.tsv`; quick plots/report in `mapping/inference/` (emu_slash_report.md + PNGs).

## Alternate database run (UNITE ITS)
- Command:
  ```bash
  mamba activate funcall
  nextflow run sixtran/main.nf \
    --db /home/uca/funcall/mapping/refdbs/unit/emu_db \
    --reads "/home/uca/funcall/oven/slashing/filtered/*.fastq" \
    --outdir /home/uca/funcall/mapping/out/emu_unite \
    --cpus 8 \
    -ansi-log false
  ```
- Outputs: `mapping/out/emu_unite/` per-sample `emu_abundance.tsv` plus combined table.

## To-do scaffolding
- Port the sixteen Nextflow `workflows/emu-nf` pipeline here (18S DB path, fungi naming).  
- Add collapser scripts (analysis/trace thresholds, OTU95 comparative + per-sample).  
- Wire a “cannon_fungi/” scratch layout parallel to `rout_fungi/`.  
- Add QC/reporting notebooks or scripts mirroring sixteen’s Rspace if needed for fungi.

## Morris inference helpers (CLR → heatmap → co-occurrence)
- Scripts (under `mapping/morris/inference/`):
  - `clr_transform.py` – pivot EMU tables to sample×feature and CLR-transform.
  - `plot_heatmap.py` – clustered heatmap of the CLR matrix (seaborn clustermap).
  - `build_cooccurrence.py` – Pearson/Spearman feature correlations → edge list + node stats.
- Default morris outputs land in `networkan/morris/`; snipe-target runs land in `networkan/snipemorris/`.
- Example (morris full set):
  ```bash
  mamba activate funcall
  python mapping/morris/inference/clr_transform.py \
    --emu-tsv mapping/morris/out/emu_morris/emu_abundance.tsv \
    --out networkan/morris/clr/clr_matrix.tsv \
    --sample-stats networkan/morris/clr/sample_stats.tsv
  python mapping/morris/inference/plot_heatmap.py \
    --clr networkan/morris/clr/clr_matrix.tsv \
    --top-n 50 \
    --out networkan/morris/heatmaps/clr_heatmap.png
  python mapping/morris/inference/build_cooccurrence.py \
    --clr networkan/morris/clr/clr_matrix.tsv \
    --method spearman --threshold 0.5 \
    --out-edges networkan/morris/networks/cooccurrence_edges.tsv \
    --out-node-stats networkan/morris/networks/node_stats.tsv
  ```
- Example (snipe-target set):
  ```bash
  mamba activate funcall
  python mapping/morris/inference/clr_transform.py \
    --emu-tsv oven/snipe/target/morris/emu_abundance.tsv \
    --out networkan/snipemorris/clr/clr_matrix.tsv \
    --sample-stats networkan/snipemorris/clr/sample_stats.tsv
  python mapping/morris/inference/plot_heatmap.py \
    --clr networkan/snipemorris/clr/clr_matrix.tsv \
    --out networkan/snipemorris/heatmaps/clr_heatmap.png
  python mapping/morris/inference/build_cooccurrence.py \
    --clr networkan/snipemorris/clr/clr_matrix.tsv \
    --method spearman --threshold 0.5 \
    --out-edges networkan/snipemorris/networks/cooccurrence_edges.tsv \
    --out-node-stats networkan/snipemorris/networks/node_stats.tsv
  ```
