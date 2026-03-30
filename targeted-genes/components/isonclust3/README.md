# IsONclust3 — Read Clustering

**Tool:** isONclust3 (ONT-aware long-read clustering)

Clusters mapped reads (per target gene, per barcode) into groups of similar sequences. Each cluster produces a per-cluster FASTQ submitted to Medaka for consensus polishing.

**Mode used:** per-target — clusters each `<gene>/<barcode>.fastq` produced by the mapping stage.

**Key parameters (`config/components/isonclust3.yml`):**

```yaml
isonclust3:
  input_mode: per-target
  threads: 8
  mode: ont
  extra: "--post-cluster"
```

**Results:** 48 clusters across all barcodes and target genes (clcA, catB2, BpHc, ntDAa).
