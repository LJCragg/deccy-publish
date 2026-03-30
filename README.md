# deccy repo — supporting codebase for MSc thesis LJS

**Microbial community structure and functional gene diversity in TCDD-contaminated soils**

MSc (Microbiology) thesis — Massey University, New Zealand

---

## Study Overview

This repository contains analysis outputs and figures from a metabarcoding study of microbial communities in TCDD-contaminated soils. Three amplicon sequencing approaches were used to characterise bacterial diversity, fungal diversity, and functional gene diversity associated with chlorophenol degradation.

Samples represent two soil cohorts (SS1 and SS2), each sequenced in triplicate (16S rRNA and ITS/18S) or as duplicate technical replicates (targeted genes) using Oxford Nanopore Technology (MinION).

---
## Current status
This repo is currently very bare bones, most notably its missing most scripts. Its mainly intended for reviewing of primary outputs and verifying data points. This is primarily a security concern, i want to verify my code is not a risk before sharing. furthermore i intend to make the code dockerized and in general reproducible. Please dont hesitstae to contact me if you want the scripts while in dev.

## Repository Structure

| Directory | Contents |
|-----------|----------|
| `16S/` | 16S rRNA bacterial amplicon analysis — OTU tables, diversity outputs, filtered results |
| `fungi/` | Fungal ITS/18S amplicon analysis — OTU tables, filtered results |
| `targeted-genes/` | Targeted functional gene amplicon analysis (clcA, catB2, BpHc, ntDAa) — consensus sequences, alignment FASTAs, coverage tables |
| `r_analysis/` | Thesis figures (PNG) and summary tables (TSV/CSV) |

---

## Data Availability

Raw sequencing data (POD5 / FASTQ) are not included in this repository. Data are available on request pending NCBI SRA submission.

---

## License

[CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/) — free to use and adapt with attribution; commercial use prohibited.

## Contact

Luca Scragg — Massey University, lucajacob1@gmail.com
