# Epigenomics & DNA Methylation Study
## Epigenetic Biomarker Analysis: Cancer Biology & Biological Aging

![Field](https://img.shields.io/badge/Field-Epigenomics-blue)
![Platform](https://img.shields.io/badge/Platform-Galaxy%20%7C%20Python-green)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)
![Course](https://img.shields.io/badge/Course-Special%20Topics%20in%20Bioinformatics-orange)

---

## Table of Contents
- [Project Overview](#project-overview)
- [Biological Background](#biological-background)
- [Repository Structure](#repository-structure)
- [Part 1: WGBS Pipeline](#part-1-wgbs-pipeline)
- [Part 2: Aging Clocks Analysis](#part-2-aging-clocks-analysis)
- [Key Results Summary](#key-results-summary)
- [Conceptual Connection Between Parts](#conceptual-connection-between-parts)
- [Requirements & Setup](#requirements--setup)
- [How to Run](#how-to-run)
- [References](#references)
- [Author](#author)

---

## Project Overview

This repository contains a two-part investigation of **DNA methylation** as a powerful epigenetic biomarker. DNA methylation is the addition of a methyl group to cytosine bases at CpG dinucleotides, playing central roles in gene regulation, cellular identity, disease, and the biology of aging. This project explores it through two independent but biologically complementary analyses:

| Feature | Part 1: WGBS Pipeline | Part 2: EPIC Array (Aging Clocks) |
|---------|----------------------|-----------------------------------|
| **Method** | Whole Genome Bisulfite Sequencing | Illumina 450K Methylation Array |
| **Platform** | Galaxy Europe (usegalaxy.eu) | Python / Google Colab |
| **Tools Used** | Falco, bwameth, MethylDackel, deepTools | Biolearn Python Library |
| **Dataset** | Lin et al. 2015 — breast cancer WGBS | GEO: GSE40279, GSE87571 |
| **Biological Focus** | Cancer-associated methylation alterations | Epigenetic aging clock predictions |
| **Key Outputs** | QC reports, alignment files, coverage plots | Correlation matrices, age heatmaps, scatter plots |

---

## Biological Background

### What is DNA Methylation?
DNA methylation is an epigenetic modification where a methyl (CH3) group is covalently attached to the 5th carbon of a cytosine base, predominantly at **CpG dinucleotides**. It is heritable through cell divisions and maintained by DNA methyltransferases (DNMTs). It plays critical roles in:

- Transcriptional gene silencing and activation
- X-chromosome inactivation and genomic imprinting
- Developmental programming
- Cancer pathogenesis
- Biological aging

### Why Does it Matter as a Biomarker?

| Context | Mechanism | Application |
|---------|-----------|-------------|
| **Cancer** | Aberrant methylation silences tumor suppressor genes and activates oncogenes | Tumor classification, early detection |
| **Aging** | Methylation at specific CpGs changes predictably with age | Epigenetic age estimation, mortality prediction |
| **Clinical** | Stable in blood, detectable at low cost | Liquid biopsy, population screening |

### Key Concepts

**Bisulfite Conversion:** In WGBS, sodium bisulfite converts unmethylated cytosines to uracil (read as thymine), while methylated cytosines remain unchanged. This enables single-base resolution methylation mapping across the genome.

**Epigenetic Clocks:** Mathematical models that combine DNA methylation beta-values at a small set of CpG sites to predict biological age. Different clocks capture different aspects of aging biology — chronological age, phenotypic age, or the pace of aging.

---

## Repository Structure

```
Epigenomics-Methylation-Study/
|
|-- README.md                               <- You are here
|
|-- Part1_WGBS_Pipeline/
|   |-- README.md                           <- Detailed Part 1 documentation
|   |-- workflow/
|   |   `-- pipeline_workflow.md            <- Step-by-step Galaxy pipeline guide
|   `-- results/
|       |-- falco_reports/
|       |   |-- falco_subset1_report.html   <- QC report sample 1
|       |   `-- falco_subset2_report.html   <- QC report sample 2
|       |-- methylation_bias_plots/
|       |   `-- README.md                   <- Bias plot description
|       |-- methylation_extraction/
|       |   `-- methylation_levels.tabular  <- Extracted CpG methylation levels
|       `-- visualization/
|           |-- plotProfile_CpG_islands.png <- Coverage plot around CpG islands
|           `-- methylation_heatmap_data.tabular
|
|-- Part2_AgingClocks_Biolearn/
|   |-- README.md                           <- Detailed Part 2 documentation
|   |-- requirements.txt                    <- Python dependencies
|   |-- notebooks/
|   |   `-- epigenetic_aging_analysis.ipynb <- Main analysis notebook
|   |-- data/
|   |   |-- predictions_dataset1_GSE40279.csv
|   |   |-- predictions_dataset2_GSE87571.csv
|   |   |-- metrics_dataset1_GSE40279.csv
|   |   `-- metrics_dataset2_GSE87571.csv
|   `-- results/
|       |-- correlation_matrix_dataset1.png
|       |-- correlation_matrix_dataset2.png
|       |-- age_deviation_heatmap_dataset1.png
|       |-- age_deviation_heatmap_dataset2.png
|       |-- scatter_plots_dataset1.png
|       `-- scatter_plots_dataset2.png
|
`-- docs/
    `-- analysis_report.md                  <- Full written project report
```

---

## Part 1: WGBS Pipeline

### Overview
Whole Genome Bisulfite Sequencing (WGBS) analysis of breast cancer methylation data, executed on [Galaxy Europe](https://usegalaxy.eu) following the Galaxy Training Network (GTN) methylation-seq tutorial.

### Dataset
- **Source:** Lin et al. 2015 (PLOS ONE) — Zenodo DOI: 10.5281/zenodo.557099
- **Comparison:** Breast tumor samples (BT089, BT126, BT198) vs Normal breast tissue (NB1, NB2)
- **Reference Genome:** hg19 / GRCh37

### Analysis Pipeline

| Step | Tool | Purpose | Output |
|------|------|---------|--------|
| 1 | Data Upload | Load FASTQ files from GTN library | subset_1.fastq, subset_2.fastq |
| 2 | **Falco** v1.3.0 | Quality control (bisulfite mode) | QC HTML reports |
| 3 | **bwameth** | Bisulfite-aware alignment to hg19 | BAM alignment files |
| 4 | **MethylDackel** (mbias) | Methylation bias detection | SVG bias plots |
| 5 | **MethylDackel** (extract) | CpG methylation level extraction | bedGraph format |
| 6 | **bamCoverage** v3.5.4 | BAM to bigWig conversion (1x normalized) | bigWig coverage file |
| 7 | **computeMatrix** v3.5.4 | Build matrix around CpG islands | Matrix file |
| 8 | **plotProfile** v3.5.4 | Visualize coverage at CpG islands | PNG profile plot |

### Key Findings
- Low %GC (~26%) confirms successful bisulfite conversion (C to T)
- 458,003 high-quality reads per sample (Phred Q35+)
- No methylation bias detected — no read trimming required
- Clear coverage enrichment at CpG island centers in plotProfile
- Methylation levels extracted for chromosomes 1, 10, and 11

---

## Part 2: Aging Clocks Analysis

### Overview
Epigenetic aging clock analysis using the **[Biolearn](https://bio-learn.github.io/)** Python library on two large publicly available GEO datasets of whole blood DNA methylation.

### Datasets Used

| GEO ID | Study | Tissue | Samples | Age Range | Platform |
|--------|-------|--------|---------|-----------|----------|
| GSE40279 | Hannum et al. 2013 | Whole blood | 656 | 19-101 yrs | Illumina 450K |
| GSE87571 | Johansson et al. 2017 | Whole blood | 729 | Wide range | Illumina 450K |

### Aging Clocks Applied

| Clock | Published | CpG Sites | What It Measures |
|-------|-----------|-----------|-----------------|
| **Horvathv1** | 2013 | 353 | Chronological age (pan-tissue) |
| **Hannum** | 2013 | 71 | Chronological age (blood-specific) |
| **PhenoAge** | 2018 | 513 | Phenotypic/biological age, linked to mortality |
| **Lin** | 2016 | 99 | Blood-based chronological age |
| **DunedinPACE** | 2022 | 173 | Pace of biological aging (rate, not absolute age) |

### Tasks Completed

| Task | Description |
|------|-------------|
| **Task 1** | Loaded 2 complete GEO datasets using Biolearn DataLibrary |
| **Task 2** | Ran all 5 aging clocks on both datasets |
| **Task 3** | Described all datasets and clocks in biological detail |
| **Task 4** | Generated Pearson correlation matrices across clock pairs |
| **Task 5** | Generated age deviation heatmaps (predicted minus chronological age) |
| **Task 6** | Generated scatter plots of predicted vs chronological age per clock |

### Key Findings
- Traditional clocks (Horvathv1, Hannum, PhenoAge, Lin) are highly intercorrelated (r = 0.89-0.99)
- DunedinPACE shows lower correlations (r = 0.42-0.58) — it measures the *rate* of aging, not absolute age
- GSE87571 shows higher inter-clock correlation than GSE40279
- All traditional clock predictions are statistically significant (p < 0.001)

---

## Key Results Summary

### Part 1 — QC Summary

| Sample | Total Reads | %GC | Per-Base Quality | Per-Sequence Quality |
|--------|------------|-----|-----------------|---------------------|
| subset_1 | 458,003 | 26.1% | PASS | PASS |
| subset_2 | 458,003 | 26.3% | PASS | PASS |

### Part 2 — Clock Performance on GSE40279

| Clock | Pearson r | MAE (years) | Interpretation |
|-------|-----------|-------------|----------------|
| Horvathv1 | ~0.92 | ~5.2 | Strong age predictor |
| Hannum | ~0.93 | ~4.8 | Best performance (blood-trained) |
| PhenoAge | ~0.91 | ~6.1 | Captures biological aging beyond chronological |
| Lin | ~0.90 | ~5.5 | Consistent blood predictor |
| DunedinPACE | ~0.48 | N/A | Measures pace (units are not years) |

---

## Conceptual Connection Between Parts

```
DNA Methylation as a Universal Epigenetic Biomarker
|
|-- Part 1: Methylation in DISEASE
|   `-- Cancer: Aberrant methylation patterns
|       -> Tumor suppressor silencing
|       -> Oncogene activation
|       Studied with WGBS at single-base resolution
|
`-- Part 2: Methylation in AGING
    `-- Epigenetic clocks: Methylation encodes biological age
        -> Predictable, reproducible, cross-tissue signal
        -> Multiple aspects: absolute age, phenotypic age, pace
        Studied with EPIC 450K arrays across large cohorts
```

Both analyses demonstrate that DNA methylation is a **stable, versatile, and clinically informative epigenetic biomarker** — applicable from cancer diagnostics to aging research.

---

## Requirements & Setup

### Part 1 — Galaxy (No installation needed)
- Create a free account at [Galaxy Europe](https://usegalaxy.eu)
- Follow the [GTN Methylation-seq Tutorial](https://training.galaxyproject.org/training-material/topics/epigenetics/tutorials/methylation-seq/tutorial.html)

### Part 2 — Python

```bash
pip install biolearn pandas numpy matplotlib seaborn scipy
# Or
pip install -r Part2_AgingClocks_Biolearn/requirements.txt
```

**Recommended:** Google Colab — no local installation needed.

---

## How to Run

### Part 2 — Local

```bash
# Clone this repository
git clone https://github.com/YOUR_USERNAME/Epigenomics-Methylation-Study.git
cd Epigenomics-Methylation-Study/Part2_AgingClocks_Biolearn

# Install dependencies
pip install -r requirements.txt

# Launch the notebook
jupyter notebook notebooks/epigenetic_aging_analysis.ipynb
```

### Part 2 — Google Colab (Recommended)
1. Go to [colab.google.com](https://colab.google.com)
2. Upload `notebooks/epigenetic_aging_analysis.ipynb`
3. Run all cells from top to bottom
4. All result plots and CSV files will be generated automatically

---

## References

1. **Lin et al. (2015).** Hierarchical Clustering of Breast Cancer Methylomes Revealed Differentially Methylated Long Non-coding RNAs. *PLOS ONE.*
2. **Ying et al. (2023).** Biolearn, an open-source library for biomarkers of aging. *bioRxiv.* https://doi.org/10.1101/2023.12.02.569722
3. **Hannum et al. (2013).** Genome-wide Methylation Profiles Reveal Quantitative Views of Human Aging Rates. *Molecular Cell.*
4. **Horvath S. (2013).** DNA methylation age of human tissues and cell types. *Genome Biology.*
5. **Levine et al. (2018).** An epigenetic biomarker of aging for lifespan and healthspan. *Aging.*
6. **Belsky et al. (2022).** DunedinPACE, a DNA methylation biomarker of the pace of aging. *eLife.*
7. **Johansson et al. (2017).** Continuous aging of the human DNA methylome throughout the human lifespan. *PLOS ONE.*

---

## Author

**Fakhruddin**  
Bioinformatics Student  
Special Topics in Bioinformatics — Course Assignment

---

## Acknowledgements
- [Galaxy Training Network (GTN)](https://training.galaxyproject.org/) for the WGBS tutorial and shared data
- [Biolearn development team](https://bio-learn.github.io/) for the open-source aging clocks library
- [NCBI GEO Database](https://www.ncbi.nlm.nih.gov/geo/) for open-access methylation datasets
