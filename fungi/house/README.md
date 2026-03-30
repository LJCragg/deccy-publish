# Basecalling

**Tool:** Dorado 1.3.0, HAC model (`dna_r10.4.1_e8.2_400bps_hac@v4.3.0`)
**Hardware:** GPU-accelerated (NVIDIA RTX 3060 Ti, CUDA)
**Kit:** SQK-NBD114-24 (native barcoding)

Basecalling and demultiplexing were performed in a single Dorado pass, retaining basecaller barcode assignments to minimise unclassified reads.

| Metric | Value |
|--------|-------|
| Input reads (POD5) | 848,367 simplex |
| Failed QC | 31 (0.004%) |
| Runtime (HAC, GPU) | ~41 minutes |
| Mean read length | 5.0–5.6 kb |

**Demultiplexed output:**

| Barcode | Reads |
|---------|-------|
| barcode19 | 101,944 |
| barcode20 | 248,391 |
| barcode21 | 433,973 |
| unclassified | 65,375 |

Raw reads and basecalling outputs are not included in this repository (gitignored).
