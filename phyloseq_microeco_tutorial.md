---
layout: default
title: 16S Phyloseq & Microeco Analysis Tutorial
---

# 🧬 16S rRNA Data Analysis in R using Phyloseq and Microeco

This tutorial provides a complete R-based workflow for analyzing 16S rRNA sequencing data using `qiime2R`, `phyloseq`, and `microeco` packages.

---

## 1️⃣ User-defined Settings

The script begins by defining working directories, filenames, and project-specific variables such as:
- `WORKDIR`: the directory for your project
- `FEATURE_TABLE_QZA`, `TAXONOMY_QZA`, `TREE_QZA`: your Qiime2 `.qza` files
- `RAREFACTION_DEPTH`: depth for rarefaction (you must update this manually)
- `TREATMENT_VAR`: name of your grouping variable (e.g. "Treatment")

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

