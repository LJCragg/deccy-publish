# MAFFT — Multiple Sequence Alignment

**Tool:** MAFFT (multiple sequence alignment)

Aligns Medaka consensus sequences per target gene, grouped by gene identity (assigned via minimap2 against reference targets). Short sequences (< 650 bp) are aligned separately as fragments.

**Key parameters (`config/components/mafft.yml`):**

```yaml
mafft:
  group_by: gene
  min_length: 650
  include_targets: true
  add_fragments: true
  minimap_preset: asm5
```

**Results:**

| Gene | Sequences aligned | Alignment length | Gap rate |
|------|-------------------|-----------------|----------|
| clcA | 7 consensus + reference | 2,458 bp (full); 576 bp (trimmed conserved region) | 72.2% → 22.8% after trimming |
| catB2 | 6 consensus + reference | — | — |
| BpHc | marginal (< threshold) | — | — |
| ntDAa | 0 (no reads recovered) | — | — |

Aligned FASTAs are in [`../../outputs/mafft/`](../../outputs/mafft/). Trimmed clcA alignment: [`../../outputs/alignment_notes/clca_trimmed.fasta`](../../outputs/alignment_notes/clca_trimmed.fasta).
