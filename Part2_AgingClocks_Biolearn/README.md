# Part 2: Epigenetic Aging Clocks Analysis (Biolearn)

## Overview
Epigenetic aging clock analysis using the **[Biolearn](https://bio-learn.github.io/)** Python library, applied to two large publicly available GEO datasets of whole blood DNA methylation profiled on the Illumina 450K array. This analysis runs five established aging clocks, compares their biological interpretations, and generates comprehensive visualizations of age predictions, correlations, and deviations.

**Reference:** Ying et al. (2023). Biolearn, an open-source library for biomarkers of aging. *bioRxiv.* https://doi.org/10.1101/2023.12.02.569722

---

## Platform
- **Google Colab:** https://colab.google.com (recommended — free, no setup)
- **Language:** Python 3
- **Core Library:** Biolearn

---

## Biological Background

### What are Epigenetic Aging Clocks?
Epigenetic aging clocks are machine learning models trained to predict chronological or biological age from DNA methylation patterns at specific CpG sites. The key insight is that methylation at certain genomic positions changes with age in a highly reproducible, tissue-consistent manner — so consistent that a small set of CpGs can estimate age to within ~5 years.

Different clocks were designed to capture different biological constructs:

| Construct | Description | Clock Example |
|-----------|-------------|---------------|
| Chronological age | Your actual calendar age | Horvathv1, Hannum |
| Biological/Phenotypic age | How old your body functions relative to peers | PhenoAge |
| Pace of aging | How fast you are aging (rate, not state) | DunedinPACE |

---

## Datasets Used

### GSE40279 — Hannum et al. 2013
- **GEO link:** https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE40279
- **Tissue:** Whole blood
- **Samples:** 656 individuals
- **Age range:** 19-101 years
- **Platform:** Illumina Infinium HumanMethylation450 BeadChip
- **Significance:** This is the landmark study used to build the Hannum aging clock. It covers the full human lifespan with healthy individuals and represents the gold standard for blood methylation aging studies.

### GSE87571 — Johansson et al. 2017
- **GEO link:** https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE87571
- **Tissue:** Whole blood
- **Samples:** 729 individuals
- **Age range:** Wide range
- **Platform:** Illumina Infinium HumanMethylation450 BeadChip
- **Significance:** A large population cohort study examining age-related methylation changes across the genome, particularly useful for validating aging biomarkers across independent samples.

---

## Aging Clocks Applied

### 1. Horvathv1 (2013)
- **Authors:** Steve Horvath, UCLA
- **CpG Sites:** 353
- **Tissue:** Pan-tissue (multi-tissue)
- **Published in:** *Genome Biology*
- **What it measures:** Chronological age across a wide variety of tissues and cell types
- **Strength:** Most widely validated, works across tissues; the "gold standard" first-generation clock

### 2. Hannum (2013)
- **Authors:** Gregory Hannum et al.
- **CpG Sites:** 71
- **Tissue:** Whole blood
- **Published in:** *Molecular Cell*
- **What it measures:** Chronological age from blood methylation
- **Strength:** Blood-specific, fewer CpGs needed, excellent performance in blood samples

### 3. PhenoAge (2018)
- **Authors:** Morgan Levine et al.
- **CpG Sites:** 513
- **Tissue:** Blood
- **Published in:** *Aging*
- **What it measures:** Phenotypic/biological age — trained on a composite of clinical biomarkers linked to mortality, not just calendar age
- **Strength:** Predicts mortality and morbidity beyond chronological age; captures accelerated aging

### 4. Lin (2016)
- **Authors:** Lin et al.
- **CpG Sites:** 99
- **Tissue:** Blood
- **Published in:** *Aging*
- **What it measures:** Chronological age from blood
- **Strength:** Compact model with solid blood-age correlation

### 5. DunedinPACE (2022)
- **Authors:** Daniel Belsky et al.
- **CpG Sites:** 173
- **Tissue:** Blood
- **Published in:** *eLife*
- **What it measures:** The *pace* of biological aging — how fast you are accumulating damage per year (output is a rate, not an age in years)
- **Strength:** More sensitive to lifestyle interventions and exposures; conceptually distinct from other clocks

---

## Tasks Completed

### Task 1: Load Datasets
Using Biolearn's `DataLibrary`, both GEO datasets were loaded directly by accession number. The methylation matrix (beta values), sample metadata, and age annotations were verified for completeness.

### Task 2: Run Aging Clocks
All 5 clocks were applied to both datasets. Output handling was adapted for DunedinPACE (pace units) vs traditional clocks (age in years).

### Task 3: Describe Datasets and Clocks
Comprehensive biological descriptions were written for each dataset (tissue type, sample size, age range, significance) and each clock (CpG count, tissue, what it measures, key strength).

### Task 4: Pearson Correlation Matrix
A Pearson correlation matrix was computed across all clock predictions for each dataset separately. Visualized as annotated heatmaps using seaborn. Reveals biological clustering of clocks by construct type.

### Task 5: Age Deviation Heatmap
Age deviation = Predicted Age minus Chronological Age, computed per sample per clock. Samples sorted by chronological age. Visualized as a heatmap to reveal systematic over- or under-prediction patterns by clock.

### Task 6: Scatter Plots
Predicted vs Chronological Age scatter plots generated for each clock and each dataset. Each subplot includes:
- Data points (color-coded by clock)
- Linear regression trend line
- Identity line (y = x, perfect prediction)
- Pearson r and p-value annotations

---

## Key Results

### Correlation Matrix Findings

| Clock Pair | GSE40279 r | GSE87571 r |
|-----------|-----------|-----------|
| Horvathv1 - Hannum | ~0.92 | ~0.99 |
| Horvathv1 - PhenoAge | ~0.91 | ~0.98 |
| Hannum - PhenoAge | ~0.93 | ~0.99 |
| Any traditional - DunedinPACE | ~0.42-0.58 | ~0.50-0.62 |

**Interpretation:** High correlations among traditional clocks confirm they all capture the same underlying DNA methylation aging signal. DunedinPACE is biologically distinct — it measures the rate of change, not the accumulated state.

### Age Deviation Findings
- Traditional clocks show balanced positive and negative deviations across samples — expected behavior for well-calibrated age predictors
- DunedinPACE consistently shows large negative values when deviation is computed — because DunedinPACE outputs values around 1.0 (pace per year), not years of age
- GSE87571 shows slightly tighter deviation patterns, consistent with its higher inter-clock correlations

### Scatter Plot Findings
- All traditional clocks: Pearson r = 0.90-0.99, p < 0.001
- DunedinPACE: Lower r (~0.48) — expected and biologically meaningful
- Hannum shows the tightest fit in blood datasets (trained on blood)

---

## Results Files

| File | Description |
|------|-------------|
| `results/correlation_matrix_dataset1.png` | Pearson correlation heatmap of all clocks — GSE40279 |
| `results/correlation_matrix_dataset2.png` | Pearson correlation heatmap of all clocks — GSE87571 |
| `results/age_deviation_heatmap_dataset1.png` | Age deviation heatmap (predicted minus chronological) — GSE40279 |
| `results/age_deviation_heatmap_dataset2.png` | Age deviation heatmap (predicted minus chronological) — GSE87571 |
| `results/scatter_plots_dataset1.png` | Predicted vs chronological age scatter plots — GSE40279 |
| `results/scatter_plots_dataset2.png` | Predicted vs chronological age scatter plots — GSE87571 |
| `data/predictions_dataset1_GSE40279.csv` | Raw clock predictions per sample — GSE40279 |
| `data/predictions_dataset2_GSE87571.csv` | Raw clock predictions per sample — GSE87571 |
| `data/metrics_dataset1_GSE40279.csv` | Pearson r and MAE summary metrics — GSE40279 |
| `data/metrics_dataset2_GSE87571.csv` | Pearson r and MAE summary metrics — GSE87571 |

---

## How to Run

### Option A: Google Colab (Recommended)
1. Go to [colab.google.com](https://colab.google.com)
2. Click **File > Upload Notebook**
3. Upload `notebooks/epigenetic_aging_analysis.ipynb`
4. Click **Runtime > Run All**
5. All plots and CSVs are generated and available for download

### Option B: Local Jupyter
```bash
pip install -r requirements.txt
jupyter notebook notebooks/epigenetic_aging_analysis.ipynb
```

---

## Requirements

```
biolearn
pandas
numpy
matplotlib
seaborn
scipy
```

Install with:
```bash
pip install biolearn pandas numpy matplotlib seaborn scipy
```

---

## References
1. Ying et al. (2023). Biolearn. *bioRxiv.*
2. Hannum et al. (2013). *Molecular Cell.*
3. Horvath S. (2013). *Genome Biology.*
4. Levine et al. (2018). *Aging.*
5. Belsky et al. (2022). *eLife.*
6. Johansson et al. (2017). *PLOS ONE.*
