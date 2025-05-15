---
layout: default
title: 16S Phyloseq & Microeco Analysis Tutorial
---

# 🧬 16S rRNA Data Analysis in R using Phyloseq and Microeco

This tutorial provides a complete R-based workflow for analyzing 16S rRNA sequencing data using `qiime2R`, `phyloseq`, and `microeco` packages.

---

## 1️⃣ User-defined Settings

The script begins by defining working directories, filenames, and project-specific variables.
<div style="position: relative; margin-bottom: 1em;">
  <pre style="background:#f6f8fa; padding:1em; border-radius:6px; overflow:auto;">
<code 
# Seed for reproducibility
MY_SEED <- 2345

# Working directory
WORKDIR <- "/home/sharmay1/data/16S/L22-UNE/R"

# Excel file containing sample metadata
METADATA_EXCEL <- "sample_metadata.xlsx"
METADATA_SHEET <- "Sheet1"  # The sheet name where metadata is stored

# Names of Qiime2 artifacts to import
FEATURE_TABLE_QZA <- "table-2.qza"
TAXONOMY_QZA      <- "taxonomy.qza"
TREE_QZA          <- "tree.qza"

# Optional .RData output file name
PSEQ_RDATA <- "XXX_pseq.RData"

# Directories and filenames for microeco step
MECO_DIR     <- "./meco"

# Rarefaction depth for microeco

RAREFACTION_DEPTH <-    #you need to come back here later

# For alpha/beta diversity, bar plots, etc.
TREATMENT_VAR <- "Treatment"   # The name of the variable in your sample_data

# File name to save final workspace & session info
FINAL_WORKSPACE <- "XXXX_final_workspace.RData"
SESSION_INFO    <- "XXXX_sessionInfo.txt"

</code>
  </pre>
  <button onclick="copyCode('bash-example')" style="
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

---

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

