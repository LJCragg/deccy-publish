# slashing

Purpose: single-step trim + filter wrapper that runs Porechop then applies the Draftpick fixed-threshold gate. Outputs length/quality-bounded FASTQs ready for mapping.

Contract
- inputs: raw FASTQ from `slashing.input_dir/slashing.pattern`
- outputs:
  - `results/slashing/{sample}.slashed.fastq`
  - `results/slashing/qc/{sample}.pre.json` and `.post.json`
- entry_rule: `slashing_all`

Behavior (defaults, override in `config/components/slashing.yml`)
- Porechop trim with optional `porechop_extra`
- Filter thresholds: min_quality=15, min_length=300, max_length=2500
- Primer requirement disabled by default; head/tail crop 15 bp before filtering

Notes
- Reuses existing Draftpick helper scripts (`components/draftpick/scripts/fastq_stats.py` and `filter_fastq.py`).
- Use `flow.postage_input: slashing` to feed Postage from the slashed FASTQs.
