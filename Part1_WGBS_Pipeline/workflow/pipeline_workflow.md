# Galaxy Pipeline Workflow — WGBS DNA Methylation Analysis
## Step-by-Step Execution Guide

**Platform:** Galaxy Europe — https://usegalaxy.eu  
**Galaxy History Name:** DNA-Methylation-WGBS  
**Tutorial Reference:** https://training.galaxyproject.org/training-material/topics/epigenetics/tutorials/methylation-seq/tutorial.html

---

## Before You Start
- Create a free account at https://usegalaxy.eu
- Log in and ensure you have storage quota available
- Use the exact tool versions listed for reproducibility

---

## Step 1: Create a New History

1. Look at the top-right of the Galaxy interface — find the **History panel**
2. Click the **"+"** (plus) button to create a new history
3. Click the history name ("Unnamed history") and rename it to: `DNA-Methylation-WGBS`
4. Press **Enter** to save

---

## Step 2: Upload the Data

1. Click **Shared Data** in the top navigation bar
2. Select **Data Libraries**
3. Navigate: **GTN-Material → Epigenetics → DNA Methylation data analysis → DOI: 10.5281/zenodo.557099**
4. Select the following files (use checkboxes):
   - `subset_1.fastq`
   - `subset_2.fastq`
   - `aligned_subset.bam`
   - `CpGIslands.bed`
5. Click **"Add to History"** → choose **"Add as Datasets to current History"**
6. Click the **blue rocket icon** to go back to your history
7. Wait for all 4 datasets to turn **green** before proceeding

---

## Step 3: Quality Control — Falco

**Why:** Verify read quality and confirm bisulfite conversion worked.

1. In the search bar (top left), type `Falco` and click the tool
2. Set parameters:
   - **Input:** Click the batch icon and select BOTH `subset_1.fastq` and `subset_2.fastq`
   - **Bisulfite Sequencing:** set to **YES**
   - Leave all other settings as default
3. Click **Execute**
4. Wait for 4 output files (RawData + Webpage for each sample) to turn green
5. Click the **eye icon** on the Webpage output to review — look for:
   - **Per Base Sequence Quality:** PASS
   - **%GC:** ~26% (normal for bisulfite data)
   - **Per Sequence Quality Scores:** PASS

---

## Step 4: Bisulfite Alignment — bwameth

**Why:** Align bisulfite-converted reads to the reference genome, accounting for C→T changes.

1. Search for `bwameth` in the tool search bar
2. Set parameters:
   - **Reference genome:** Select built-in **hg19**
   - **Single-end or paired-end:** Single-end
   - **Input file:** `subset_1.fastq` (run once for each sample)
3. Click **Execute**
4. Repeat for `subset_2.fastq`
5. Wait for BAM output files (~60 MB each) to turn green

---

## Step 5: Methylation Bias Check — MethylDackel (mbias)

**Why:** Check for position-dependent methylation artifacts at read ends that require trimming.

1. Search for `MethylDackel` and open the tool
2. Set parameters:
   - **Mode:** `Determine the position-dependent methylation bias (mbias)`
   - **Reference genome:** hg19
   - **Aligned reads (BAM):** Select the bwameth BAM output
3. Click **Execute** (run for each BAM file)
4. Check the SVG output plots — if bias plots are **flat/horizontal**, no trimming is needed
5. **In this analysis:** No bias was detected — proceed to extraction without trimming

---

## Step 6: Methylation Extraction — MethylDackel (extract)

**Why:** Extract the actual methylation percentage at every CpG site covered by reads.

1. Open **MethylDackel** again
2. Set parameters:
   - **Mode:** `Extract methylation metrics (extract)`
   - **Reference genome:** hg19
   - **Aligned reads (BAM):** Use `aligned_subset.bam` (precomputed library file — not your bwameth output)
   - Leave trimming settings at 0 (no bias detected in Step 5)
3. Click **Execute**
4. Output is a bedGraph file — rows represent individual CpG sites with methylation percentage

---

## Step 7: BAM to bigWig — bamCoverage

**Why:** Convert BAM alignment to bigWig format for visualization and computeMatrix input.

1. Search for `bamCoverage` and open the tool
2. Set parameters:
   - **BAM file:** `aligned_subset.bam`
   - **Output format:** bigwig
   - **Normalization method:** 1x (RPGC)
   - **Effective genome size:** Use `hg19/GRCh37` option (or enter 2685511504)
   - **Bin size:** 50
3. Click **Execute**
4. Output: bigWig file (~4.7 MB)

---

## Step 8: Build Coverage Matrix — computeMatrix

**Why:** Calculate coverage values in a window around CpG island centers for aggregated visualization.

1. Search for `computeMatrix` and open the tool
2. Set parameters:
   - **Regions (BED/GTF):** `CpGIslands.bed`
   - **Score file (bigWig):** bamCoverage output from Step 7
   - **Mode:** `reference-point`
   - **Reference point:** Center of region
   - **Distance upstream:** 500
   - **Distance downstream:** 500
3. Click **Execute**

---

## Step 9: Plot Coverage Profile — plotProfile

**Why:** Visualize coverage enrichment around CpG island centers as an aggregate line plot.

1. Search for `plotProfile` and open the tool
2. Set parameters:
   - **Input:** computeMatrix output from Step 8
   - Output format: PNG
3. Click **Execute**
4. Click the **eye icon** on the PNG output
5. You should see a characteristic **enrichment peak at the center** (position 0 = CpG island center)

---

## Tools Used and Versions

| Step | Tool | Version |
|------|------|---------|
| QC | Falco | 1.3.0+galaxy0 |
| Alignment | bwameth | latest |
| Bias check | MethylDackel | 0.5.2+galaxy0 |
| Extraction | MethylDackel | 0.5.2+galaxy0 |
| Coverage | bamCoverage | 3.5.4+galaxy0 |
| Matrix | computeMatrix | 3.5.4+galaxy0 |
| Plot | plotProfile | 3.5.4+galaxy0 |

---

## Common Issues and Fixes

| Problem | Likely Cause | Fix |
|---------|-------------|-----|
| High %GC in Falco | Bisulfite mode not enabled | Re-run Falco with bisulfite=YES |
| bwameth error | Wrong genome (hg38 vs hg19) | Select hg19 specifically |
| computeMatrix error | BAM not indexed | Use the BAI file from the library |
| plotProfile shows flat line | Wrong bigWig file selected | Re-check bamCoverage output |

---

## Important Notes

- Always use **hg19** (not hg38) — all files are aligned to hg19
- For MethylDackel extraction, use the **pre-aligned BAM from the library** (`aligned_subset.bam`), not your bwameth output — this ensures coverage across more chromosomes
- Enable **bisulfite mode** in Falco or results will be misleading
- Galaxy jobs may take 5-15 minutes to complete — wait for green before proceeding
