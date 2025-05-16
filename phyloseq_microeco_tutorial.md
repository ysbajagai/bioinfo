---
layout: default
title: 16S Phyloseq & Microeco Analysis Tutorial
---

# 🧬 16S rRNA Data Analysis in R using Phyloseq and Microeco

This tutorial provides a complete R-based workflow for analyzing 16S rRNA sequencing data using `qiime2R`, `phyloseq`, and `microeco` packages.

---

## 1️⃣ User-defined Settings
The script begins by defining working directories, filenames, and project-specific variables.

---
<div style="position: relative; margin-bottom: 1em;">
  <pre style="background:#f6f8fa; padding:1em; border-radius:6px; overflow:auto;">
<code id="r-setup" style="font-family: monospace;">
### Seed for reproducibility
MY_SEED &lt;- 2345
### Working directory
WORKDIR &lt;- "XXXXXXX"
### Excel file containing sample metadata
METADATA_EXCEL &lt;- "sample_metadata.xlsx"
METADATA_SHEET &lt;- "Sheet1"  # The sheet name where metadata is stored
### Names of Qiime2 artifacts to import
FEATURE_TABLE_QZA &lt;- "table-2.qza"
TAXONOMY_QZA      &lt;- "taxonomy.qza"
TREE_QZA          &lt;- "tree.qza"
### Optional .RData output file name
PSEQ_RDATA &lt;- "XXX_pseq.RData"
### Directories and filenames for microeco step
MECO_DIR &lt;- "./meco"
### Rarefaction depth for microeco
RAREFACTION_DEPTH &lt;-   # you need to come back here later
### For alpha/beta diversity, bar plots, etc.
TREATMENT_VAR &lt;- "Treatment"   # The name of the variable in your sample_data
### File name to save final workspace & session info
FINAL_WORKSPACE &lt;- "XXXX_final_workspace.RData"
SESSION_INFO    &lt;- "XXXX_sessionInfo.txt"
</code>
  </pre>
  <button onclick="copyCode('r-setup')" style="
    position: absolute;
    top: 10px;
    right: 10px;
    background-color: #0366d6;
    color: white;
    border: none;
    padding: 4px 8px;
    border-radius: 5px;
    font-size: 0.8em;
    cursor: pointer;">📋 Copy</button>
</div>

<script>
function copyCode(id) {
  const code = document.getElementById(id).innerText;
  navigator.clipboard.writeText(code).then(() => {
    alert("✅ Code copied to clipboard!");
  });
}
</script>
---
## 2️⃣ Install and Load Required Packages
The script installs (if necessary) and loads:
- CRAN packages: `ggplot2`, `dplyr`, `readxl`, `paletteer`, `microeco`
- Bioconductor: `phyloseq`
- GitHub: `qiime2R`
  
### A) cran_packages
<div style="position: relative; margin-bottom: 1em;">
  <pre style="background:#f6f8fa; padding:1em; border-radius:6px; overflow:auto;">
<code id="r-packages" style="font-family: monospace;">
cran_packages &lt;- c(
  "readxl",
  "dplyr",
  "tibble",
  "ggplot2",
  "RColorBrewer",
  "paletteer",
  "file2meco",
  "microeco",
  "GUniFrac",    # for UniFrac distances
  "vegan"        # often needed for ordination or diversity calculations
)
missing_cran &lt;- cran_packages[!(cran_packages %in% installed.packages()[,"Package"])]
if(length(missing_cran)) install.packages(missing_cran)
### B) Bioconductor packages (phyloseq, etc.)
if (!requireNamespace("BiocManager", quietly = TRUE))
  install.packages("BiocManager")

if (!"phyloseq" %in% installed.packages()[,"Package"]) {
  BiocManager::install("phyloseq")
}
### C) GitHub package for qiime2R
if (!"qiime2R" %in% installed.packages()[,"Package"]) {
  if (!requireNamespace("devtools", quietly = TRUE))
    install.packages("devtools")
  devtools::install_github("jbisanz/qiime2R")
}
### D) (Optional) microbiome package for taxa_filter()
if (!"microbiome" %in% installed.packages()[,"Package"]) {
install.packages("microbiome")
}
### Now load libraries
library(readxl)
library(tibble)
library(dplyr)
library(qiime2R)
library(phyloseq)
library(ggplot2)
library(file2meco)
library(microeco)
library(RColorBrewer)
library(paletteer)
library(GUniFrac)
library(vegan)       
library(microbiome)  
</code>
  </pre>
  <button onclick="copyCode('r-packages')" style="
    position: absolute;
    top: 10px;
    right: 10px;
    background-color: #0366d6;
    color: white;
    border: none;
    padding: 4px 8px;
    border-radius: 5px;
    font-size: 0.8em;
    cursor: pointer;">📋 Copy</button>
</div>

<script>
function copyCode(id) {
  const code = document.getElementById(id).innerText;
  navigator.clipboard.writeText(code).then(() => {
    alert("✅ Code copied to clipboard!");
  });
}
</script>
---
## Environment Setup
<div style="position: relative; margin-bottom: 1em;">
  <pre style="background:#f6f8fa; padding:1em; border-radius:6px; overflow:auto;">
<code id="r-env-setup" style="font-family: monospace;">
###############################################################################
# 2. ENVIRONMENT SETUP
###############################################################################
set.seed(MY_SEED)        # For reproducibility
rngseed &lt;- MY_SEED       # Keep track of the seed
setwd(WORKDIR)           # Set working directory
</code>
  </pre>
  <button onclick="copyCode('r-env-setup')" style="
    position: absolute;
    top: 10px;
    right: 10px;
    background-color: #0366d6;
    color: white;
    border: none;
    padding: 4px 8px;
    border-radius: 5px;
    font-size: 0.8em;
    cursor: pointer;">📋 Copy</button>
</div>

<script>
function copyCode(id) {
  const code = document.getElementById(id).innerText;
  navigator.clipboard.writeText(code).then(() => {
    alert("✅ Code copied to clipboard!");
  });
}
</script>





## 3️⃣ Import Metadata and Create a Phyloseq Object

You’ll read your metadata from Excel using `readxl`, and convert Qiime2 artifacts using `qza_to_phyloseq()`. Then merge all into a `phyloseq` object.

```r
samples_df <- read_excel(METADATA_EXCEL, sheet = METADATA_SHEET)
otu_mat <- qza_to_phyloseq(features = FEATURE_TABLE_QZA)
pseq <- phyloseq(otu_mat, tax_mat, samples, tree)
```

---

## 4️⃣ Filter Taxa

Remove:
- Unassigned or ambiguous taxa
- Mitochondria, Chloroplasts
- Eukaryota

Optional filtering by minimum frequency is commented.

---

## 5️⃣ Visualize Tree and Abundance

You’ll prune top N taxa and visualize with:
```r
plot_tree(ex1, color = TREATMENT_VAR)
```

---

## 6️⃣ Microeco Analysis

- Convert `phyloseq` → `microeco`
- Rarefy to `RAREFACTION_DEPTH`
- Compute and save:
  - Taxonomic abundance
  - Alpha & beta diversity (UniFrac, PD, Chao1, etc.)
  - Barplots by group or sample

Plots saved as both `.pdf` and `.png`.

---

## 7️⃣ Alpha & Beta Diversity

- `trans_alpha$new()` calculates richness metrics like Shannon, Simpson
- `trans_beta$new()` computes ordinations like PCoA, NMDS with UniFrac

Boxplots and ordination plots are saved in your output directory.

---

## 8️⃣ Differential Abundance (LEfSe & Metastat)

- LEfSe for multiple taxonomic levels using `trans_diff$new(...)`
- Metastat also included as an alternate method
- Results exported as CSV files
- Try-catch blocks ensure the script continues even if some tax ranks fail

---

## 9️⃣ Final Save

```r
save.image(file = FINAL_WORKSPACE)
writeLines(capture.output(sessionInfo()), SESSION_INFO)
```

---

## 📦 Download

This page accompanies a script for your analysis. To get the full R script and required file structure, [please contact the tutorial maintainer].

