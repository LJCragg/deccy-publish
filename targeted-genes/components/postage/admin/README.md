# postage (integrated)

Purpose: targeted mapping/validation using existing Postage rules and scripts.

Contract
- inputs: FASTQ (trimmed or raw per `config/roadtrip.yml`), reference FASTA
- outputs: BAM, coverage tables, abundance tables under `results/postage/{sample}/`
- entry_rule: `postage:all`

Notes
- See original design: `postage/postage_admin_postage_design_document.md` in the playground.
- We'll align output paths and config names to roadtrip conventions.
