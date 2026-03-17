# Funcall basecalling workflow

## Monorepo path note

This pipeline now lives under `/home/uca/chover/funcall`. If you copy the commands below verbatim, replace `/home/uca/funcall` with `/home/uca/chover/funcall`.

- **Input**: `input/Fungi/20230303_1429_MC-115797_FAX53729_8bbd279f/pod5_pass/`
- **Outputs (latest run)**:
  - BAMs: `output/dorado_basecall_cuda/071124_Dioxin_soils_2/Fungi/20230303_1429_MC-115797_FAX53729_8bbd279f/bam_pass/`
  - FASTQs (flattened, basecaller assignments kept): `output/noclass/fastqpass/` (files: `barcode19.fastq`, `barcode20.fastq`, `barcode21.fastq`, `unclassified.fastq`)
  - QC: `output/noclass/qc/` (`fastq_summary.tsv`, `reads_bases_bars.png`, `length_hist.png`)
- **Tools**: Dorado 1.3.0 (HAC model `dna_r10.4.1_e8.2_400bps_hac@v4.3.0`), GPU RTX 3060 Ti.

## Commands used (GPU)
```bash
MODEL="/home/uca/funcall/dordowna/dorado-1.3.0-linux-x64/models/dna_r10.4.1_e8.2_400bps_hac@v4.3.0"
POD5="/home/uca/funcall/house/input/Fungi/20230303_1429_MC-115797_FAX53729_8bbd279f/pod5_pass"
OUT="/home/uca/funcall/house/output/dorado_basecall_cuda"

./dordowna/dorado-1.3.0-linux-x64/bin/dorado basecaller \
  --device cuda:0 --recursive \
  --kit-name SQK-NBD114-24 \
  --emit-summary \
  --output-dir "$OUT" \
  "$MODEL" "$POD5"

DEMUX_OUT="/home/uca/funcall/house/output/dorado_demux"
./dordowna/dorado-1.3.0-linux-x64/bin/dorado demux \
  --recursive \
  --kit-name SQK-NBD114-24 \
  --emit-fastq \
  --output-dir "$DEMUX_OUT" \
  "$OUT"
```

## Quick results snapshot (current)
- Basecalling: 848,367 reads (31 filtered) in ~41 min on RTX 3060 Ti (HAC model).
- Demux (basecaller barcode calls retained; non-target barcodes removed):
  - barcode21: 433,973
  - barcode20: 248,391
  - barcode19: 101,944
  - unclassified: 65,375
- QC highlights: mean/median length (bp) — 21: 4,998/5,166; 20: 5,316/5,177; 19: 5,589/5,166; unclassified: 5,202/5,243; all N fractions 0%.
- Demuxed FASTQs live under `output/noclass/fastqpass/`; paths are now flattened.

## Fungal prefilter (“slashing”)
- Location: `oven/slashing/` (filtered FASTQs under `filtered/`, QC tables in `qc_before/`, `qc_after/`, summary in `summary.tsv`).
- Filters: chopper with min length 1,000 bp, max length 8,000 bp, min mean Q 9.
- Rerun (from repo root):  
  ```bash
  mamba activate funcall
  mkdir -p oven/slashing/{filtered,qc_before,qc_after,logs}
  seqkit stats -a house/output/noclass/fastqpass/*.fastq > oven/slashing/qc_before/seqkit_stats.tsv
  for f in house/output/noclass/fastqpass/*.fastq; do base=$(basename "$f"); chopper -q 9 -l 1000 --maxlength 8000 < "$f" > "oven/slashing/filtered/$base" 2> "oven/slashing/logs/$base.log"; done
  seqkit stats -a oven/slashing/filtered/*.fastq > oven/slashing/qc_after/seqkit_stats.tsv
  python oven/slashing/summarize_qc.py
  ```

## Fungal mapping context (Dec 1 2025)
- Combined MIMt 18S + ITS EMU DB built at `mapping/refdbs/mergemimt/emu_db` (taxdump: `mapping/refdbs/mergemimt/ncbisrc/taxdump/`).
- Normalised reference inputs: `mimt_clean.filtered.fasta` (61,924 seqs) + `mimt_seq2tax.tsv` (headerless seq→taxid).
- Runtime env (Nextflow + EMU): `sixtran/funcall.yml` → `mamba activate funcall`.
- Nextflow fungal pipeline entry: `/home/uca/funcall/sixtran/main.nf` (set `--db` to the EMU DB above).

## 16S turtle (APW124 + AQA415) — demux snapshot (Dec 8 2025)
- Basecaller: Dorado 1.3.0 HAC (`dna_r10.4.1_e8.2_400bps_hac@v4.3.0`); kit applied at demux.
- **Basecaller assignments kept** (`--no-classify --no-trim`): merged FASTQs in `output/dorado_basecall_cuda/16swork/200825_Luca_turtle/fastq_merged/`; counts mostly unclassified (395,933/395,980 reads).
- **Kit-classified** (`--kit-name SQK-NBD114-24`): merged FASTQs in `output/dorado_basecall_cuda/16swork/200825_Luca_turtle/fastq_merged_classify/`; counts: bc01 80,905; bc02 80,744; bc03 55,634; bc04 54,488; bc06 12,885; unclassified 111,313; others ≤2 reads (total 397,684).
- QC summaries live alongside the merged FASTQs (`qc_summary.md`, git-ignored with outputs); provenance captured in `house/admin/2025-12-08_16s_turtle_demux.md`.
