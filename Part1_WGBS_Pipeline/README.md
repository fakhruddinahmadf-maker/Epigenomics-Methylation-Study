# Part 1: WGBS DNA Methylation Pipeline (Galaxy)

## Overview
Whole Genome Bisulfite Sequencing (WGBS) analysis of breast cancer methylation data, executed step-by-step on the [Galaxy Europe](https://usegalaxy.eu) cloud bioinformatics platform, following the Galaxy Training Network (GTN) methylation-seq tutorial.

**Reference:** Lin et al. (2015). Hierarchical Clustering of Breast Cancer Methylomes Revealed Differentially Methylated Long Non-coding RNAs. *PLOS ONE.*

---

## Platform & Resources
- **Galaxy Europe:** https://usegalaxy.eu
- **History Name Used:** DNA-Methylation-WGBS
- **Tutorial Reference:** https://training.galaxyproject.org/training-material/topics/epigenetics/tutorials/methylation-seq/tutorial.html

---

## Biological Context

Breast cancer frequently displays widespread aberrant DNA methylation — particularly hypermethylation of tumor suppressor gene promoters and global hypomethylation. WGBS enables genome-wide, single-base resolution mapping of these methylation changes, which is critical for:
- Identifying differentially methylated regions (DMRs) between tumor and normal tissue
- Discovering novel methylated long non-coding RNAs (lncRNAs)
- Understanding epigenetic drivers of oncogenesis

---

## Dataset Details

| Item | Description |
|------|-------------|
| **Source** | Lin et al. 2015, Zenodo DOI: 10.5281/zenodo.557099 |
| **Input FASTQ** | subset_1.fastq, subset_2.fastq |
| **Pre-aligned BAM** | aligned_subset.bam |
| **Reference genome** | hg19 (GRCh37) |
| **Tumor samples** | BT089, BT126, BT198 |
| **Normal samples** | NB1, NB2 |

---

## Step-by-Step Pipeline

### Step 1: Data Upload
- Navigate to **Shared Data > Data Libraries > GTN-Material > Epigenetics**
- Select: **DNA Methylation data analysis > DOI: 10.5281/zenodo.557099**
- Add subset_1.fastq, subset_2.fastq, aligned_subset.bam, CpGIslands.bed to history

---

### Step 2: Quality Control — Falco
**Tool:** Falco v1.3.0+galaxy0

Falco performs FastQC-style quality assessment with native support for bisulfite-sequenced data.

| Parameter | Setting |
|-----------|---------|
| Input | subset_1.fastq AND subset_2.fastq (batch) |
| Bisulfite Sequencing | YES (enabled) |
| Output | RawData + HTML Webpage per file |

**Results:**

| Sample | Total Reads | %GC | Per Base Quality | Per Sequence Quality |
|--------|------------|-----|-----------------|---------------------|
| subset_1 | 458,003 | 26.1% | PASS | PASS |
| subset_2 | 458,003 | 26.3% | PASS | PASS |

**Why low %GC is expected:** Bisulfite treatment converts unmethylated cytosines (C) to uracil/thymine (T), dramatically reducing overall GC content from the typical ~40-50% to ~26%. This confirms successful conversion.

---

### Step 3: Alignment — bwameth
**Tool:** bwameth

bwameth is a bisulfite-aware aligner that correctly handles C-to-T converted reads by using a 3-letter genome alignment strategy.

| Parameter | Setting |
|-----------|---------|
| Reference genome | Built-in hg19 |
| Sequencing type | Single-end |
| Input | subset_1.fastq, subset_2.fastq |
| Output | BAM alignment files (~60 MB each) |

---

### Step 4: Methylation Bias — MethylDackel (mbias mode)
**Tool:** MethylDackel v0.5.2+galaxy0

Detects position-dependent methylation bias caused by end-repair artifacts, which can falsely inflate methylation at read ends.

| Parameter | Setting |
|-----------|---------|
| Mode | Determine methylation bias (mbias) |
| Reference | hg19 |
| Input | bwameth BAM outputs |
| Output | SVG bias plots for all 4 strands (OT, OB, CTOT, CTOB) |

**Result:** No significant position-dependent bias detected. No read trimming required before extraction.

---

### Step 5: Methylation Extraction — MethylDackel (extract mode)
**Tool:** MethylDackel v0.5.2+galaxy0

Extracts CpG methylation levels for every covered cytosine in the genome.

| Parameter | Setting |
|-----------|---------|
| Mode | Extract methylation metrics |
| Reference | hg19 |
| Input | aligned_subset.bam (precomputed, from library) |
| Output | CpG methylation levels in bedGraph format |
| Coverage | chr1, chr10, chr11 |

**Output format:** Each line = one CpG site with chromosome, position, strand, coverage, and methylation percentage.

---

### Step 6: bamCoverage (BAM to bigWig)
**Tool:** bamCoverage v3.5.4

Converts BAM alignment file to bigWig format for genome browser visualization and downstream matrix computation.

| Parameter | Setting |
|-----------|---------|
| Input | aligned_subset.bam |
| Output format | bigwig |
| Normalization | 1x (RPGC — reads per genomic content) |
| Genome size | hg19 effective size: 2,685,511,504 |
| Bin size | 50 bp |

---

### Step 7: computeMatrix
**Tool:** computeMatrix v3.5.4

Builds a coverage matrix centered on genomic features (CpG islands) for aggregated visualization.

| Parameter | Setting |
|-----------|---------|
| Regions BED file | CpGIslands.bed |
| Score file | bamCoverage bigwig output |
| Mode | reference-point |
| Reference point | center of region |
| Upstream distance | 500 bp |
| Downstream distance | 500 bp |

---

### Step 8: plotProfile
**Tool:** plotProfile v3.5.4

Generates an aggregated coverage profile plot around CpG island centers.

| Parameter | Setting |
|-----------|---------|
| Input | computeMatrix output |
| Output | PNG profile plot |

**Result:** Clear enrichment peak visible at CpG island centers, confirming successful analysis and expected methylation patterns.

---

## Results Files

| File | Description |
|------|-------------|
| `results/falco_reports/falco_subset1_report.html` | Full QC report for sample 1 (open in browser) |
| `results/falco_reports/falco_subset2_report.html` | Full QC report for sample 2 (open in browser) |
| `results/visualization/plotProfile_CpG_islands.png` | Coverage enrichment profile around CpG islands |
| `results/visualization/methylation_heatmap_data.tabular` | Heatmap matrix data |
| `results/methylation_extraction/methylation_levels.tabular` | CpG site methylation percentages |

---

## Key Findings Summary

1. **Bisulfite conversion confirmed** — Low %GC (~26%) in both samples
2. **High read quality** — Tight Phred Q35+ distribution across read length
3. **Clean alignment** — BAM files ~60 MB per sample
4. **No bias correction needed** — MethylDackel mbias showed flat methylation across read positions
5. **CpG island enrichment confirmed** — plotProfile shows characteristic peak at island centers
6. **Methylation data extracted** — bedGraph files ready for downstream DMR analysis
