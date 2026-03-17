# RACECAR: R Analysis & Charting Environment

**A fully portable, RStudio-ready analysis workspace for EMURE-processed metabarcoding data**

---

## Overview

The `racecar` project is a portable R analysis workspace for exploring and
visualizing 16S bacterial and ITS fungal metabarcoding data derived from the
current EMURE-era pipeline outputs. This directory contains bundled input
tables, scripts, and utilities for generating thesis-facing figures and summary
statistics for the dioxin-contaminated soil microbiome study.

**Key Features:**
- ✅ Fully portable: all input data are real files (no symbolic links)
- ✅ RStudio project (.Rproj) configured and ready
- ✅ Consistent ggplot2 theming via `theme_race()`
- ✅ Bundled input copies for portable downstream analysis
- ✅ Organized outputs by analysis scope (sixteen/funcall)
- ✅ Export set consumed by the top-level `writing/` packet workflow

---

## Project Structure

```
racecar/
├── race.Rproj                    # RStudio project file
├── README.md                     # This file
├── components/                   # Standalone analysis components
│   └── sixteen_denovo_emubased_discussion/
│       ├── README.md             # Discussion-focused de-novo vs Emu based comparison
│       └── scripts/
│           └── run_comparison.R  # Canonical comparison against claude/sixteen outputs
├── scripts/                      # All R analysis scripts
│   ├── utils.R                   # Shared utilities, loaders, themes
│   ├── 01_sixteen_overview.R     # 16S: Overview plots
│   ├── 02_sixteen_genus_trends.R # 16S: Genus-level trends
│   ├── 03_sixteen_diversity_pcoa.R   # 16S: Alpha/beta diversity + PCoA
│   ├── 04_sixteen_dioxin_heatmap.R   # 16S: Dioxin-degrader heatmap
│   ├── 05_sixteen_read_length.R      # 16S: Read length (disabled)
│   ├── 06_funcall_emure_overview.R   # Fungal: EMU overview
│   ├── 07_sixteen_otu97_transition.R # 16S: OTU97 transitions
│   ├── 08_funcall_otu95_overview.R   # Fungal: OTU95 overview
│   ├── 09_funcall_threepoint_analysis.R  # Fungal: Threepoint analysis
│   ├── 10_sixteen_network.R          # 16S: Co-occurrence networks
│   ├── 11_funcall_shroom_filter.R    # Fungal: RETIRED
│   └── 12_funcall_diversity.R        # Fungal: Alpha diversity
├── inputs/                       # All input data (real files, fully portable)
│   ├── sixteen/                  # 16S bacterial data
│   │   ├── emu_abundance.tsv              # EMU species-level abundances
│   │   ├── emure_otu97_prethreepoint.tsv  # OTU97 pre-filter table
│   │   ├── emure_otu97_threepoint.tsv     # OTU97 threepoint-filtered
│   │   ├── emure_associated_threepoint.tsv # Associated degrader OTUs
│   │   └── long_associated_emu.tsv        # Associated EMU abundances (long)
│   └── funcall/                  # ITS fungal data (fungi-filtered upstream)
│       ├── emure_associated_threepoint.tsv         # Associated OTUs (genus-level)
│       ├── emure_associated_family_threepoint.tsv  # Associated OTUs (family-level)
│       ├── emure_associated_phylum_threepoint.tsv  # Associated OTUs (phylum-level)
│       ├── emure_otu95_threepoint.tsv              # OTU95 threepoint-filtered
│       ├── fungi_emure_prethree.tsv                # OTU95 pre-threepoint (long)
│       └── funcall_raw_emu_abundance.tsv           # Raw EMU abundances (all taxa)
├── outputs/                      # Generated data tables & RDS plot objects
│   ├── sixteen/
│   │   ├── threepoint/           # All-OTU analyses (diversity, transitions)
│   │   ├── associated/           # Dioxin-degrader whitelist analyses
│   │   ├── network/              # Co-occurrence network analyses
│   │   └── plots/                # RDS plot objects (same subcategories)
│   │       ├── threepoint/
│   │       ├── associated/
│   │       └── network/
│   └── funcall/
│       ├── threepoint/           # All-OTU analyses (OTU95, diversity)
│       ├── associated/           # Dioxin-degrader whitelist analyses
│       └── plots/                # RDS plot objects (same subcategories)
│           ├── threepoint/
│           └── associated/
├── figures/                      # Generated plots (PNG, 300 dpi)
│   ├── sixteen/
│   │   ├── threepoint/           # All-OTU figures
│   │   ├── associated/           # Degrader-focused figures
│   │   └── network/              # Network analysis figures
│   └── funcall/
│       ├── threepoint/           # All-OTU figures
│       └── associated/           # Degrader-focused figures
└── archive/                      # Legacy scripts and outputs

```

---

## Input Data

All input files in `inputs/` are **real copies** (not symbolic links), making this project fully portable and self-contained. The data includes:

### 16S Bacterial Metabarcoding (`inputs/sixteen/`)

| File | Description |
|------|-------------|
| `emu_abundance.tsv` | EMU species-level abundance table (284K) |
| `emure_otu97_prethreepoint.tsv` | OTU97 table before threepoint filtering (50K) |
| `emure_otu97_threepoint.tsv` | OTU97 table after threepoint filtering (37K) |
| `emure_associated_threepoint.tsv` | Degrader-associated OTUs passing threepoint (5.6K) |
| `long_associated_emu.tsv` | EMU abundances for associated taxa (long format, 43K) |

### ITS Fungal Metabarcoding (`inputs/funcall/`)

Fungal inputs are **pre-filtered upstream** before OTU clustering — non-fungal eukaryotes
(plants, animals, nematodes) were removed at the read level prior to vsearch clustering.
Chlorophyta (green algae) are intentionally retained. The raw EMU file is unfiltered for
contamination assessment.

| File | Description |
|------|-------------|
| `emure_otu95_threepoint.tsv` | OTU95 table after threepoint filtering (91 OTUs, wide format) |
| `fungi_emure_prethree.tsv` | OTU95 pre-threepoint table (642 rows, collapser long format) |
| `emure_associated_threepoint.tsv` | Associated fungal OTUs — genus-level target list (4 OTUs) |
| `emure_associated_family_threepoint.tsv` | Associated fungal OTUs — family-level target list (12 OTUs) |
| `emure_associated_phylum_threepoint.tsv` | Associated fungal OTUs — phylum-level target list (62 OTUs) |
| `funcall_raw_emu_abundance.tsv` | Raw EMU abundance table — all taxa incl. non-fungal (1305 rows) |

**Threepoint Filtering:** A quality control step requiring OTUs to be:
1. Present in ≥3 samples, OR
2. Present in ≥2 samples with ≥0.5% mean relative abundance, OR  
3. Present in ≥1 sample with ≥1% relative abundance

---

## Analysis Scripts

All scripts are designed to be run from the `racecar/` root directory.

### Usage Pattern

```r
# Open the RStudio project first
# File → Open Project → race.Rproj

# Then run any script:
source("scripts/01_sixteen_overview.R")

# Or from terminal:
cd /path/to/cousin/racecar
Rscript scripts/01_sixteen_overview.R
```

### Script Inventory

#### 16S Bacterial Analyses

| Script | Purpose | Key Outputs |
|--------|---------|-------------|
| `01_sixteen_overview.R` | Overview plots of associated OTUs | Genus bar charts, abundance distributions |
| `02_sixteen_genus_trends.R` | Genus-level abundance trends across samples | Faceted genus plots by cohort |
| `03_sixteen_diversity_pcoa.R` | Alpha/beta diversity + PCoA ordination | PCoA plots, distance matrices, alpha tables |
| `04_sixteen_dioxin_heatmap.R` | Heatmap of dioxin-degrader abundances | Clustered heatmaps |
| `05_sixteen_read_length.R` | Read length distribution (disabled) | N/A |
| `07_sixteen_otu97_transition.R` | OTU97 filtering transition summary | Before/after comparison tables |
| `10_sixteen_network.R` | Co-occurrence network analysis | Network graphs, modules, degrader subnet |

#### ITS Fungal Analyses

| Script | Purpose | Key Outputs |
|--------|---------|-------------|
| `06_funcall_emure_overview.R` | Fungal EMU overview (filtered to fungi) | Top genera bar charts |
| `08_funcall_otu95_overview.R` | OTU95 cluster distribution | OTU abundance, broad-group, richness plots |
| `09_funcall_threepoint_analysis.R` | Threepoint & associated OTU analysis | Heatmaps, summary tables (3 assoc. levels) |
| `11_funcall_shroom_filter.R` | **RETIRED** — filtering now upstream | N/A |
| `12_funcall_diversity.R` | Fungal alpha/beta diversity (OTU95) | Alpha diversity and Bray-Curtis tables |

#### Shared Utilities

- **`utils.R`**: Core utilities loaded by all scripts
  - Data loading functions with EMURE-first path resolution
  - Sample/cohort labeling functions
  - `theme_race()` ggplot2 theme
  - Path management utilities

#### Standalone Components

- `components/sixteen_denovo_emubased_discussion/scripts/run_comparison.R`
  - Discussion-specific comparison of the canonical de-novo and Emu based 16S OTU tables before and after threepoint filtering
  - Writes figures and tables under `discussion_denovo_emubased/`
  - Reads directly from `sixteen/discussion/denovo/` and `sixteen/cannon/emure/`

---

## Sample Metadata

The project bundles two marker datasets with different sample designs:

### 16S bacterial data

| Cohort | Barcodes | Sample IDs | Description |
|--------|----------|------------|-------------|
| **Ss1** | BC13-15 | barcode13_120_filtered<br>barcode14_120_filtered<br>barcode15_120_filtered | Soil sample 1 replicates |
| **Ss2** | BC16-18 | barcode16_120_filtered<br>barcode17_120_filtered<br>barcode18_120_filtered | Soil sample 2 replicates |

### Fungal ITS/18S data

| Marker | Barcodes | Sample IDs | Description |
|--------|----------|------------|-------------|
| **ITS/18S** | BC19-21 | barcode19<br>barcode20<br>barcode21 | Fungal amplicon libraries |

Fungal barcodes BC19-21 should not be relabeled as the 16S Ss1/Ss2 cohorts.

---

## Dependencies

### Required R Packages

The following packages are loaded by `scripts/utils.R`:

```r
# Core tidyverse
library(ggplot2)    # Data visualization
library(dplyr)      # Data manipulation
library(tidyr)      # Data tidying
library(readr)      # Data import
library(stringr)    # String operations
library(forcats)    # Factor handling
library(tibble)     # Modern data frames
library(purrr)      # Functional programming

# Additional utilities
library(scales)     # Scale functions for ggplot2
```

### Installation

If any packages are missing, install them with:

```r
install.packages(c("ggplot2", "dplyr", "tidyr", "readr", 
                   "stringr", "forcats", "tibble", "purrr", "scales"))

# Or install the entire tidyverse:
install.packages("tidyverse")
```

---

## Key Functions

### Data Loading (`utils.R`)

All data loading functions follow an **EMURE-first** path resolution policy, checking EMURE pipeline outputs before falling back to legacy paths:

#### 16S Loaders
- `load_emu()` – EMU species-level abundances
- `load_otu97_raw()` – OTU97 pre-threepoint table
- `load_otu97_threepoint()` – OTU97 post-threepoint table
- `load_associated97_long()` – Associated degrader OTUs (long format)
- `load_associated_emu()` – Associated EMU abundances
- `load_associated_long()` – OTU95 associated (legacy, long format)
- `load_associated_wide()` – OTU95 associated (legacy, wide format)
- `load_associated_summary()` – OTU95 associated summary statistics
- `load_threepoint()` – OTU95 threepoint table
- `load_core()` – OTU95 core microbiome table
- `load_target_genera()` – Dioxin-degrader genera reference list

#### Fungal Loaders
- `load_funcall_emu()` – Fungal EMU abundances
- `load_funcall_otu95()` – Fungal OTU95 cluster table
- `load_funcall_otu95_long()` – Fungal OTU95 long format
- `load_funcall_otu95_threepoint()` – Fungal OTU95 threepoint-filtered
- `load_funcall_otu95_threepoint_summary()` – Fungal threepoint summary
- `load_funcall_associated()` – Fungal associated OTUs (long)
- `load_funcall_associated_summary()` – Fungal associated summary

### Utilities

- `cohort_from_sample(sample_id)` – Extract cohort (Ss1/Ss2) from sample ID
- `short_sample(sample_id)` – Remove `_120_filtered` suffix for display
- `cohort_label_long(cohort)` – Format cohort labels with barcode ranges
- `set_output_scope(scope)` – Set output directories ("sixteen" or "funcall")
- `ensure_dir(...)` – Create output directories if they don't exist
- `theme_race(base_size = 11)` – Consistent ggplot2 theme for all plots

---

## Output Organization

Outputs are organized by **scope** (sixteen / funcall) and **category** (threepoint / associated / network):

| Category | Description |
|----------|-------------|
| `threepoint` | Analyses on all OTUs that pass threepoint prevalence filtering |
| `associated` | Analyses restricted to the *a priori* dioxin-degrader whitelist |
| `network` | Co-occurrence network analyses (16S only) |

```r
# set_output_scope() configures these base directories:
# FIG_DIR  <- "figures/sixteen"   or  "figures/funcall"
# OUT_DIR  <- "outputs/sixteen"   or  "outputs/funcall"
# PLOT_DIR <- "outputs/sixteen/plots"  or  "outputs/funcall/plots"

set_output_scope("sixteen")  # For 16S analyses
set_output_scope("funcall")  # For fungal analyses
```

Within each scope, pass a `category` to route files into sub-directories:

```r
save_fig("fig09_alpha_diversity.png", p, category = "threepoint")
# → figures/sixteen/threepoint/fig09_alpha_diversity.png

save_table("tbl06_associated_evidence_summary.tsv", df, category = "associated")
# → outputs/sixteen/associated/tbl06_associated_evidence_summary.tsv

save_plot_obj("fig14_network_genus.rds", p, category = "network")
# → outputs/sixteen/plots/network/fig14_network_genus.rds
```

### Interactive Plot Viewer

Use `P()` to browse saved RDS plot objects interactively:

```r
P()         # List all saved plots (grouped by category)
P(13)       # Display fig13 (found recursively across categories)
P("network") # Display first plot matching "network"
```

Figures are saved as PNG (300 dpi). Data tables are saved as TSV.

## Thesis Integration

`racecar` no longer stores thesis-revision files. Rewrite drafts, agent context, and writing notes live under `../waffle/`.

Figures and tables generated here feed directly into thesis drafts via the file paths documented in `../waffle/agent/RACE_CONTEXT.md`.

---

## Workflow Example

```r
# 1. Open the RStudio project
# File → Open Project → race.Rproj

# 2. Run 16S overview analysis
source("scripts/01_sixteen_overview.R")
# → Generates: figures/sixteen/associated/*.png
#              figures/sixteen/threepoint/*.png
#              outputs/sixteen/plots/associated/*.rds
#              outputs/sixteen/plots/threepoint/*.rds

# 3. Run fungal EMU analysis
source("scripts/06_funcall_emure_overview.R")
# → Generates: figures/funcall/threepoint/*.png
#              outputs/funcall/threepoint/*.tsv

# 4. Or run all scripts in sequence
scripts <- list.files("scripts", pattern = "^\\d{2}_.*\\.R$", full.names = TRUE)
lapply(scripts, source)
```

---

## Data Provenance

`racecar` uses bundled copies of the current thesis-facing input tables under
`inputs/`. Upstream processing remains canonical in:

- `sixteen/` (canonical EMURE outputs in `sixteen/cannon/emure/`)
- `funcall/`
- `roadtrip/`

`racecar` should be treated as the portable downstream analysis and thesis
assembly layer, not as the raw-processing source of truth.

---

## Portability Notes

This directory is **fully portable** and can be:
- ✅ Copied to any system
- ✅ Shared via USB drive or cloud storage
- ✅ Archived without dependencies on parent directories
- ✅ Opened directly in RStudio on any platform

All input data are **real files** (not symbolic links), ensuring:
- No broken links when moving directories
- No dependencies on other workspace locations
- Complete self-contained analysis environment

---

## Troubleshooting

### Missing Packages

```r
# Check if a package is installed
if (!require("ggplot2")) install.packages("ggplot2")

# Install all required packages at once
required <- c("ggplot2", "dplyr", "tidyr", "readr", "stringr", 
              "forcats", "tibble", "purrr", "scales")
new_pkgs <- required[!(required %in% installed.packages()[,"Package"])]
if(length(new_pkgs)) install.packages(new_pkgs)
```

### File Not Found Errors

If you encounter "File not found" errors:

1. **Check working directory**: Must be `racecar/` root
   ```r
   getwd()  # Should end in "/racecar"
   setwd("/path/to/cousin/racecar")  # If not
   ```

2. **Verify input files exist**:
   ```r
   list.files("inputs/sixteen")
   list.files("inputs/funcall")
   ```

3. **Check script source path**: Always use `scripts/filename.R`
   ```r
   source("scripts/utils.R")  # Correct
   # Not: source("utils.R")   # Wrong
   ```

### Output Directory Errors

Scripts automatically create output directories, but you can manually ensure they exist:

```r
source("scripts/utils.R")
ensure_dir("figures/sixteen", "figures/funcall", 
           "outputs/sixteen", "outputs/funcall")
```

---

## Related Projects

- **`sixteen/`** – Parent 16S bacterial metabarcoding pipeline (EMURE in `sixteen/cannon/emure/`)
- **`funcall/`** – Parent ITS fungal metabarcoding pipeline
- **`roadtrip/`** – Targeted functional gene pipeline
- **`waffle/`** – Thesis writing workspace

---

## Citation & Contact

This analysis workspace is part of an MSc thesis investigating microbial communities in dioxin-contaminated soils at the anonymized contaminated site, with a focus on dioxin-degrading bacteria and fungi identification through EMURE-based metabarcoding pipelines.

**Last Updated:** March 9, 2026

---

## Quick Start Checklist

- [ ] Open `race.Rproj` in RStudio
- [ ] Install required packages (see Dependencies section)
- [ ] Verify working directory is `racecar/`
- [ ] Run `source("scripts/utils.R")` to load utilities
- [ ] Run individual analysis scripts or batch process all scripts
- [ ] Check `figures/` and `outputs/` for results

**Ready to analyze!** 🚀
