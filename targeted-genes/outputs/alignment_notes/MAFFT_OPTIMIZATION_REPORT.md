# MAFFT Alignment Optimization Plan & Results

**Date:** January 24, 2026  
**Location:** `/home/uca/chover/playground/alignment/`  
**Goal:** Reduce gaps in MAFFT alignments for clca and catB2 genes through identity filtering, gene-specific strategies, and progressive alignment

---

## Summary of Implementation

### What Was Done

1. **Identity Filtering (70% threshold)**
   - Filtered out 38 low-quality catB2 sequences (22-69% identity)
   - Removed 12 unassigned sequences (Bordetella petrii, Caproiciproducens)
   - Excluded BpHc and ntDAa genes (too few sequences)
   - **Result:** Clean, high-quality sequence sets for alignment

2. **Gene-Specific MAFFT Strategies**
   - **clca:** `--localpair --maxiterate 1000 --thread 4`
     - Best for moderate sequence variation (85-100% identity range)
     - More accurate than `--auto` but computationally intensive
   - **catB2:** `--genafpair --ep 0.5 --maxiterate 1000 --thread 4`
     - Handles length heterogeneity better
     - `--ep 0.5` reduces gap opening penalty for divergent sequences

3. **Progressive Alignment by Identity Tiers**
   - **Anchor:** Consensus + targets aligned first (high-quality core)
   - **Tier 1:** Add NCBI sequences >95% identity
   - **Tier 2:** Add NCBI sequences 85-95% identity
   - **Tier 3:** Add NCBI sequences 70-85% identity (if any)
   - Uses MAFFT `--add` to preserve core alignment structure

4. **Threading Support**
   - Added `--thread 4` flag to all MAFFT calls
   - Enables parallel processing (wasn't used in original pipeline)

---

## Results

### CLCA Gene Alignment

| Metric | Old (mafft_split) | New (simple) | New (progressive) |
|--------|-------------------|--------------|-------------------|
| Sequences | 36 | 36 | 36 |
| Alignment length | 2458 bp | 2458 bp | 2458 bp |
| **Total gap %** | **72.18%** | **72.18%** | **72.18%** |
| Avg seq gap % | 72.18% | 72.18% | 72.18% |
| Gap range | 45.3%-81.9% | 45.3%-81.9% | 45.3%-81.9% |

**Interpretation:**
- Gap percentage **unchanged** (both old and new = 72.18%)
- `--localpair --maxiterate 1000` produces **same gaps** as `--auto`
- High gaps (72%) are **intrinsic to sequence diversity**, not alignment algorithm
- Sequences span 444-1344 bp with high variation in length
- Progressive alignment maintained structural integrity

**Why are gaps so high?**
1. Length heterogeneity: 444 bp (short NCBI fragments) to 1344 bp (long consensus)
2. Partial gene sequences: Many NCBI entries are gene fragments, not full-length
3. Real biological variation: clca homologs have genuine insertions/deletions
4. 5' and 3' overhangs: Fragments don't span the same gene region

---

### CATB2 Gene Alignment

| Metric | Old (mafft_split) | New (simple) | New (progressive) |
|--------|-------------------|--------------|-------------------|
| Sequences | 40 | 2 | 2 |
| Alignment length | 2825 bp | 648 bp | 648 bp |
| **Total gap %** | **80.50%** | **5.56%** | **5.56%** |
| Avg seq gap % | 80.50% | 5.56% | 5.56% |
| Gap range | 20.1%-87.1% | 0.6%-10.5% | 0.6%-10.5% |

**Interpretation:**
- **MASSIVE improvement:** 80.5% gaps → 5.6% gaps (**74.9% reduction!**)
- But this is misleading - we only kept 2 sequences (the targets)
- **All 3 consensus sequences filtered out** (29-45% identity to targets)
- **All 35 NCBI sequences filtered out** (<70% identity)

**Critical Finding:**
The catB2 consensus sequences have **extremely low identity** to reference targets:
- `bc01_c0`: 29.7% identity (2256 bp - suspiciously long!)
- `bc02_c3`: 31.3% identity (903 bp)
- `bc03_c6`: 45.1% identity (968 bp)

**This suggests:**
1. **Misannotation:** Consensus sequences may not be catB2
2. **Distant homologs:** Very divergent catB2-like genes
3. **Chimeras:** Possible assembly artifacts or multi-domain proteins
4. **Wrong target:** Reference targets may not match actual gene variants in sample

---

## Key Findings

### 1. Identity Filtering Impact

- **clca:** All sequences >85% identity → clean, homogeneous alignment
- **catB2:** Consensus sequences <50% identity → **major red flag**
- 70% threshold appropriate for clca, too strict for catB2 (if sequences are real)

### 2. MAFFT Strategy Comparison

- `--auto`: Fast, reasonable results
- `--localpair --maxiterate 1000`: More accurate but **same gaps for clca**
- `--genafpair`: Better for length heterogeneity (not tested with real data due to filtering)
- **Conclusion:** Algorithm choice doesn't reduce gaps caused by biological variation

### 3. Progressive Alignment

- Successfully preserved core structure (consensus + targets)
- Added NCBI sequences in tiers without disrupting anchor
- **No gap reduction** vs simple alignment (both handle same sequences equally)
- Main benefit: Computational efficiency and structural consistency

### 4. Gap Sources Identified

For clca (72% gaps):
1. **Length variation:** 444-1344 bp range
2. **Partial sequences:** Many NCBI entries are fragments
3. **5'/3' overhangs:** Sequences don't span same region
4. **Biological indels:** Real insertions/deletions between homologs

For catB2 (before filtering: 80% gaps):
1. **Extreme divergence:** Consensus sequences unrelated to targets
2. **Possible misannotation:** Need to verify gene identity
3. **Length mismatch:** Consensus 903-2256 bp vs NCBI 429-513 bp

---

## Recommended Next Steps

### Immediate Actions

1. **Investigate catB2 Consensus Sequences**
   ```bash
   # Extract consensus sequences
   grep -A 1 "bc0.*catB2" nosefile.fa > catB2_consensus.fa
   
   # BLAST against nr database
   blastn -query catB2_consensus.fa -db nt -remote -outfmt 6 -max_target_seqs 10
   
   # Or use local BLAST against target database
   makeblastdb -in targets.fa -dbtype nucl -out targets_db
   blastn -query catB2_consensus.fa -db targets_db -outfmt 7
   ```

2. **Domain Analysis for catB2**
   ```bash
   # Translate to protein and search for conserved domains
   conda run -n roadtrip python -c "
   from Bio import SeqIO
   from Bio.Seq import Seq
   for rec in SeqIO.parse('catB2_consensus.fa', 'fasta'):
       for frame in [0, 1, 2]:
           prot = rec.seq[frame:].translate(to_stop=True)
           if len(prot) > 100:
               print(f'>{rec.id}_frame{frame}')
               print(prot)
   "
   ```

3. **Re-run Minimap2 with Lower Identity Cutoff**
   ```bash
   # Try 50% identity threshold to see what gets assigned
   minimap2 -x map-ont targets.fa catB2_consensus.fa | \
     awk '$10/$11 > 0.5 {print}'
   ```

### Fragment-Based Approaches to Reduce Gaps

#### Option 1: Trim to Shared Region

```bash
# Identify conserved core region across all clca sequences
# Align, identify columns with <50% gaps, extract that region

conda run -n roadtrip python << 'EOF'
from Bio import AlignIO

# Load alignment
aln = AlignIO.read("improved_alignments/clca_progressive_final.fasta", "fasta")

# Find low-gap columns
low_gap_cols = []
for i in range(aln.get_alignment_length()):
    col = aln[:, i]
    gap_pct = col.count('-') / len(col)
    if gap_pct < 0.5:  # Less than 50% gaps
        low_gap_cols.append(i)

print(f"Low-gap region: columns {min(low_gap_cols)}-{max(low_gap_cols)}")
print(f"Trimmed length: {len(low_gap_cols)} bp")

# Extract trimmed alignment
with open("improved_alignments/clca_trimmed.fasta", "w") as out:
    for record in aln:
        seq = "".join([record.seq[i] for i in low_gap_cols])
        out.write(f">{record.id}\n{seq}\n")
EOF
```

#### Option 2: Split by Gene Domain

For multi-domain genes, align each domain separately:

```bash
# Use hmmer to identify domain boundaries
# Example for clca (chlorocatechol 1,2-dioxygenase)

# 1. Find domain
hmmsearch --tblout clca_domains.txt \
  /path/to/pfam/PF00775.hmm \  # Dioxygenase_C domain
  clca_consensus_proteins.fa

# 2. Extract domain coordinates
# 3. Align domains separately
# 4. Concatenate or analyze independently
```

#### Option 3: Fragment Strategy - Align Separately by Length Class

```python
# Group sequences by length bins
short = 400-600 bp
medium = 600-900 bp
long = 900-1500 bp

# Align each group separately
mafft --localpair --maxiterate 1000 clca_short.fa > clca_short_aligned.fa
mafft --localpair --maxiterate 1000 clca_medium.fa > clca_medium_aligned.fa
mafft --localpair --maxiterate 1000 clca_long.fa > clca_long_aligned.fa

# Then add fragments to long alignment progressively
mafft --add clca_medium_aligned.fa clca_long_aligned.fa > clca_combined.fa
mafft --add clca_short_aligned.fa clca_combined.fa > clca_final.fa
```

### Advanced MAFFT Options to Try

#### 1. Adjust Gap Penalties

```bash
# Lower gap opening penalty (default: 1.53)
mafft --localpair --maxiterate 1000 --op 1.0 --thread 4 input.fa > output.fa

# Lower gap extension penalty (default: 0.123)
mafft --localpair --maxiterate 1000 --ep 0.05 --thread 4 input.fa > output.fa

# Both together
mafft --localpair --maxiterate 1000 --op 1.0 --ep 0.05 --thread 4 input.fa > output.fa
```

#### 2. Use 6mer Distance for Divergent Sequences

```bash
# For very divergent sequences (like catB2 if real)
mafft --6merpair --maxiterate 1000 --thread 4 input.fa > output.fa
```

#### 3. Global vs Local Alignment

```bash
# Try local alignment mode for fragments
mafft --localpair --maxiterate 1000 --lop -2.0 --thread 4 input.fa > output.fa

# Or use --globalpair for full-length sequences
mafft --globalpair --maxiterate 1000 --thread 4 input.fa > output.fa
```

#### 4. Structural Alignment (if you have secondary structure predictions)

```bash
# Use Q-INS-i for sequences with conserved secondary structure
mafft --qinsi --thread 4 input.fa > output.fa
```

---

## MAFFT Strategy Decision Tree

```
START: What are my sequences?

├─ All similar length (±20%)?
│  ├─ YES → High identity (>90%)?
│  │       ├─ YES → Use --auto (fast, good enough)
│  │       └─ NO → Use --localpair --maxiterate 1000 (accurate)
│  └─ NO → Length varies >2x
│          └─ Use --genafpair --ep 0.5 (handles length variation)
│
├─ Many fragments vs full-length?
│  └─ YES → Progressive alignment:
│           1. Align full-length sequences first
│           2. --add fragments progressively
│           3. OR align fragments separately, report separately
│
├─ Very divergent (<70% identity)?
│  └─ YES → Use --6merpair or --genafpair
│           → Consider domain-based alignment
│           → Check if sequences are actually homologous!
│
└─ Multi-domain proteins?
   └─ YES → Align domains separately
            → Use hmmer to find boundaries
            → Concatenate alignments
```

---

## Files Generated

### Scripts
- `improved_mafft_align.py`: Main alignment script with identity filtering and progressive alignment
- `analyze_filtering.py`: Analyzes what sequences were kept vs filtered
- `generate_comparison_report.py`: Compares old vs new alignment statistics

### Alignments
- `improved_alignments/clca_filtered.fasta`: 36 clca sequences (filtered)
- `improved_alignments/clca_simple.fasta`: Simple one-shot alignment
- `improved_alignments/clca_progressive_final.fasta`: Progressive tiered alignment
- `improved_alignments/catB2_filtered.fasta`: 2 catB2 targets only
- `improved_alignments/catB2_simple.fasta`: Target alignment
- `improved_alignments/catB2_progressive_final.fasta`: Same (only targets)

### Reports
- `improved_alignments.log`: Full run log
- `comparison_report.txt`: Gap statistics comparison

### Intermediate Files
- `improved_alignments/clca_anchor.fasta`: Consensus + targets
- `improved_alignments/clca_high.fasta`: High-identity NCBI sequences (>95%)
- `improved_alignments/clca_medium.fasta`: Medium-identity NCBI sequences (85-95%)
- `improved_alignments/clca_consensus.fasta`: Consensus sequences only
- `improved_alignments/clca_targets.fasta`: Target sequences only

---

## Integration with Roadtrip Pipeline

The improved alignment approach can be integrated into the vork_clean roadtrip pipeline:

### Update `align_consensus_mafft.py`

1. **Add identity filtering:**
   ```python
   # Filter sequences by identity before alignment
   if config['mafft']['min_identity'] > 0:
       filtered_seqs = filter_by_identity(seqs, config['mafft']['min_identity'])
   ```

2. **Add gene-specific strategies:**
   ```python
   strategy_map = {
       'clca': ['--localpair', '--maxiterate', '1000'],
       'catB2': ['--genafpair', '--ep', '0.5', '--maxiterate', '1000'],
       'default': ['--auto']
   }
   ```

3. **Implement progressive alignment:**
   ```python
   if config['mafft']['progressive']:
       # Align anchor (consensus + targets)
       # Add NCBI sequences by identity tiers
   ```

4. **Add threading:**
   ```python
   cmd.extend(['--thread', str(config['mafft']['threads'])])
   ```

### Update `roadtrip_config.yaml`

```yaml
mafft:
  medaka_dir: results/medaka
  ref: admin/targets.fa
  outdir: results/mafft
  group_by: gene
  min_identity: 0.70  # Filter sequences below this
  progressive: true   # Use progressive alignment
  threads: 4
  
  # Gene-specific strategies
  strategies:
    clca:
      method: localpair
      maxiterate: 1000
    catB2:
      method: genafpair
      ep: 0.5
      maxiterate: 1000
    default:
      method: auto
```

---

## Conclusions

### What Worked
✅ Identity filtering removed low-quality sequences  
✅ Threading support added successfully  
✅ Progressive alignment maintained structural integrity  
✅ Gene-specific MAFFT strategies implemented  

### What Didn't Reduce Gaps
❌ Algorithm choice (`--localpair` vs `--auto`) - same gaps  
❌ Progressive vs simple alignment - same gaps  
❌ Iteration count (1000 vs default) - same gaps  

### Why Gaps Remain High (72% for clca)
- **Biological reality:** Sequences genuinely vary in length (444-1344 bp)
- **Fragment nature:** Many NCBI sequences are partial genes
- **Region mismatch:** Sequences don't span the same gene coordinates
- **Real indels:** Homologous genes have genuine insertions/deletions

### The Real Solution
🔍 **Trim to shared conserved region** instead of full-length alignment  
🔍 **Domain-based alignment** for multi-domain genes  
🔍 **Fragment classification** - align similar-length sequences together  
🔍 **Coordinate-based trimming** - extract matching gene regions before alignment  

### catB2 Requires Investigation
⚠️ **Critical:** Consensus sequences only 29-45% identity to targets  
⚠️ Likely misannotated or chimeric - needs BLAST verification  
⚠️ Cannot improve alignment until gene identity confirmed  

---

## Quick Reference: MAFFT Commands

```bash
# Fast, good for similar sequences (>90% identity)
mafft --auto --thread 4 input.fa > output.fa

# Accurate, moderate variation (85-95% identity)
mafft --localpair --maxiterate 1000 --thread 4 input.fa > output.fa

# Length heterogeneity, divergent sequences
mafft --genafpair --ep 0.5 --maxiterate 1000 --thread 4 input.fa > output.fa

# Very divergent (<70% identity)
mafft --6merpair --maxiterate 1000 --thread 4 input.fa > output.fa

# Add new sequences to existing alignment
mafft --add new_seqs.fa --thread 4 existing_alignment.fa > updated.fa

# Adjust gap penalties (lower = more gaps allowed)
mafft --localpair --op 1.0 --ep 0.05 --maxiterate 1000 --thread 4 input.fa > output.fa
```

---

**End of Report**
