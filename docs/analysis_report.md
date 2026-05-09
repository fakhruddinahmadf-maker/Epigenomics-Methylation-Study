# Epigenomics & DNA Methylation — Analysis Report
**Course:** Special Topics in Bioinformatics  
**Author:** Fakhruddin  
**Assignment:** WGBS / Biolearn Analysis

---

## Introduction

DNA methylation is a fundamental epigenetic modification in which a methyl group (–CH₃) is added to the 5th carbon of cytosine, predominantly at CpG dinucleotides. Unlike genetic mutations, methylation is a reversible, heritable modification that regulates gene expression without changing the DNA sequence. It plays critical roles in:

- Transcriptional regulation (gene silencing/activation)
- Cellular differentiation and development
- X-chromosome inactivation
- Genomic imprinting
- Cancer development and progression
- Biological aging

This project investigates DNA methylation from two complementary angles: cancer biology using Whole Genome Bisulfite Sequencing (WGBS), and aging biology using array-based epigenetic aging clocks.

---

## Part 1: WGBS Analysis

### Background

Whole Genome Bisulfite Sequencing (WGBS) is the gold standard method for genome-wide, single-base resolution mapping of DNA methylation. The method relies on chemical treatment with sodium bisulfite, which converts unmethylated cytosines to uracil (read as thymine during sequencing), while methylated cytosines remain unchanged. By comparing reads to the reference genome, every CpG site in the genome can be classified as methylated or unmethylated.

In cancer, this is particularly powerful: tumor cells often show global DNA hypomethylation alongside focal hypermethylation of tumor suppressor gene promoters — patterns that WGBS can detect with precision.

### Dataset

Breast cancer methylation data from Lin et al. (2015), comparing five breast cancer samples (BT089, BT126, BT198 — tumor) with two normal breast tissue samples (NB1, NB2). Data accessed from Zenodo DOI: 10.5281/zenodo.557099.

This dataset was chosen because breast cancer is among the best-characterized cancers at the epigenetic level, with well-documented promoter hypermethylation events affecting tumor suppressor genes like BRCA1, CDH1, and RASSF1A.

### Methods

The following pipeline was executed on Galaxy Europe (usegalaxy.eu):

**1. Quality Control (Falco):** Reads were assessed for quality using Falco in bisulfite mode. Both samples passed all QC metrics. Low %GC (~26%) is the hallmark of successful bisulfite conversion — normal genomic DNA has ~40-50% GC, but after bisulfite treatment, converted C residues (now T) reduce GC content dramatically.

**2. Alignment (bwameth):** Bisulfite-converted reads were aligned to the human reference genome (hg19) using bwameth, a tool specifically designed for bisulfite sequencing. Standard aligners fail with bisulfite data because C→T conversions create massive mismatches; bwameth accounts for this by using a 3-letter genome strategy.

**3. Methylation Bias Assessment (MethylDackel mbias):** Methylation bias was assessed across all read positions for all four strands. No significant position-dependent bias was detected, meaning end-repair artifacts were absent and no read trimming was required before extraction.

**4. Methylation Extraction (MethylDackel extract):** CpG methylation levels were extracted from the pre-aligned BAM file, producing a bedGraph file where each line represents one CpG site with its methylation percentage. Coverage was obtained for chromosomes 1, 10, and 11.

**5. Visualization (bamCoverage + computeMatrix + plotProfile):** A coverage bigWig was generated (1x RPGC normalized), then computeMatrix calculated coverage in ±500 bp windows centered on CpG islands. plotProfile generated the final aggregated coverage visualization.

### Results

All 458,003 reads in both samples passed quality control with high Phred scores (peak at Q35+). The low %GC (~26%) confirmed successful bisulfite conversion. No methylation bias was detected across read positions. The plotProfile output shows characteristic enrichment at CpG island centers, confirming the analysis pipeline functioned correctly and that aligned reads cluster at these biologically important regulatory regions.

Methylation levels were extracted for chromosomes 1, 10, and 11, providing a representative subset of genome-wide CpG methylation that can be used for downstream differential methylation analysis between tumor and normal tissue.

---

## Part 2: Aging Clocks Analysis

### Background

Epigenetic aging clocks are mathematical models that exploit the highly reproducible relationship between DNA methylation patterns and chronological age. Steve Horvath's 2013 landmark paper demonstrated that a set of just 353 CpG sites could predict age to within ~3.6 years across diverse tissue types — a finding that revealed DNA methylation as a molecular timekeeping mechanism.

Since Horvath's work, multiple clocks have been developed with different biological targets:
- Some predict pure chronological age
- Others predict biological/phenotypic age (linked to mortality)
- The newest clocks measure the pace of aging rather than age itself

These clocks have become important tools for understanding aging biology, evaluating interventions, and developing biomarkers for age-related diseases.

### Datasets

**GSE40279 (Hannum et al. 2013):** 656 whole blood samples spanning ages 19-101 years, profiled on the Illumina 450K methylation array. This is the landmark dataset used to build the Hannum aging clock and is among the most widely cited methylation aging studies.

**GSE87571 (Johansson et al. 2017):** 729 whole blood samples from a population cohort, also on the Illumina 450K array. Used to study genome-wide age-related methylation changes and validates aging biomarkers in an independent cohort.

### Methods

The Biolearn Python library was used to load datasets directly from GEO by accession number and apply aging clocks programmatically. Five clocks were applied: Horvathv1, Hannum, PhenoAge, Lin, and DunedinPACE.

**Task 4 — Correlation Matrix:** Pearson correlation coefficients were computed between all pairwise clock predictions using `scipy.stats.pearsonr` and visualized as annotated heatmaps using seaborn. This reveals which clocks capture similar vs distinct aging biology.

**Task 5 — Age Deviation Heatmap:** For each sample, age deviation was calculated as: Predicted Age minus Chronological Age. Samples were sorted by chronological age to reveal systematic over/under-prediction patterns. A diverging colormap was used so positive deviations (predicted older than actual) appear in one color and negative deviations (predicted younger) appear in another.

**Task 6 — Scatter Plots:** Predicted vs chronological age was plotted for each clock on each dataset. Each subplot includes a regression line, the ideal y=x identity line, and Pearson r with p-value to quantify prediction accuracy.

### Results

#### Correlation Matrix (Task 4)

Traditional aging clocks (Horvathv1, Hannum, PhenoAge, Lin) show high pairwise Pearson correlations (r = 0.89-0.99). This convergence confirms that despite using different CpG sites and training datasets, these clocks all capture the same underlying biological signal — methylation drift that accumulates with age in a consistent, reproducible manner across individuals.

DunedinPACE shows substantially lower correlations with traditional clocks (r = 0.42-0.58). This is expected and biologically meaningful: DunedinPACE measures the pace of aging (how fast biological damage is accumulating) rather than biological age (how much has accumulated). These are conceptually different constructs.

GSE87571 shows higher inter-clock correlations overall compared to GSE40279, suggesting a more cohesive aging pattern in that cohort, possibly due to more homogeneous sample composition.

#### Age Deviation Heatmap (Task 5)

Traditional clocks show balanced deviations — roughly equal numbers of samples predicted older and younger than their actual age. This balanced pattern indicates well-calibrated clocks that do not systematically over- or under-predict in a given direction.

DunedinPACE consistently shows large negative deviations when age deviation is computed in the traditional sense. This is a technical artifact of comparing units: DunedinPACE outputs values around 1.0 (pace units per calendar year), not years of age. Its large negative deviations reflect the difference between a value near 1.0 and a chronological age of 50-80 years — not true biological over- or under-aging.

#### Scatter Plots (Task 6)

All traditional clocks show strong linear relationships between predicted and chronological age with Pearson r = 0.90-0.99 and p < 0.001. The Hannum clock shows the tightest fit in blood datasets, consistent with it having been trained specifically on blood methylation.

DunedinPACE shows lower r (~0.48), expected given it measures a different biological construct. Nonetheless, its significant correlation with age confirms that individuals aging faster (higher DunedinPACE score) also tend to be chronologically older in these cross-sectional cohorts.

---

## Discussion

### Comparing the Two Approaches

WGBS and 450K array methylation analysis serve complementary but different purposes:

| Aspect | WGBS (Part 1) | 450K Array (Part 2) |
|--------|---------------|---------------------|
| Resolution | Single-base, genome-wide | Pre-selected ~450,000 CpGs |
| Coverage | Entire genome (~28 million CpGs) | Targeted sites |
| Cost | High | Moderate |
| Best use | Discovery, novel DMR detection, cancer | Large cohorts, established biomarkers, aging clocks |
| Sample size | Small N, deep coverage | Large N, broad population studies |

Neither approach is superior — they answer different biological questions. WGBS is ideal for cancer epigenomics where novel methylation changes must be discovered at single-base resolution. Array-based profiling is ideal for aging clocks, where validated sites are already known and large sample sizes are needed for statistical power.

### Biological Significance

The high concordance among traditional aging clocks (r > 0.89) is remarkable and meaningful — it demonstrates that the biological process of aging leaves a highly reproducible molecular signature across different individuals. This robustness is what makes epigenetic clocks practically useful as biomarkers: you can measure them once from a blood draw and obtain reliable biological age estimates.

DunedinPACE's distinct behavior highlights a conceptually important distinction in aging research between:
- **Biological age** (how old your body is at a given point — a state)
- **Pace of aging** (how fast you are aging — a rate)

A person can be biologically old but aging slowly (good prognosis), or biologically young but aging quickly (concerning prognosis). Combining both measures gives richer information than either alone.

---

## Conclusion

This project demonstrates that DNA methylation is a versatile molecular biomarker with applications across two major areas of biomedicine: cancer and aging. WGBS reveals cancer-specific methylation alterations at single-base resolution, enabling discovery of epigenetic drivers of oncogenesis. Array-based aging clock analysis demonstrates the remarkable consistency of the epigenetic aging signal across individuals and validates five established aging clocks on independent datasets.

The complementary nature of these two approaches — and the shared molecular mechanism underlying both (aberrant DNA methylation) — reinforces the central importance of epigenomics in modern biomedical research. As sequencing and array technologies become cheaper and more accessible, DNA methylation profiling will increasingly transition from a research tool to a clinical diagnostic biomarker for cancer, aging, and disease risk stratification.

---

## References

1. Lin et al. (2015). Hierarchical Clustering of Breast Cancer Methylomes. *PLOS ONE.*
2. Ying et al. (2023). Biolearn, an open-source library for biomarkers of aging. *bioRxiv.*
3. Hannum et al. (2013). Genome-wide Methylation Profiles Reveal Quantitative Views of Human Aging Rates. *Molecular Cell.*
4. Horvath S. (2013). DNA methylation age of human tissues and cell types. *Genome Biology.*
5. Levine et al. (2018). An epigenetic biomarker of aging for lifespan and healthspan. *Aging.*
6. Lu et al. (2019). DNA methylation GrimAge strongly predicts lifespan and healthspan. *Aging.*
7. Belsky et al. (2022). DunedinPACE, a DNA methylation biomarker of the pace of aging. *eLife.*
8. Johansson et al. (2017). Continuous aging of the human DNA methylome throughout the human lifespan. *PLOS ONE.*
