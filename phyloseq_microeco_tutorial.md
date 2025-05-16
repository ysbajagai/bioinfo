---
layout: default
title: 16S Phyloseq & Microeco Analysis Tutorial
---

# 16S rRNA Sequencing Data Analysis in R using Phyloseq and Microeco

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
  
<div style="position: relative; margin-bottom: 1em;">
  <pre style="background:#f6f8fa; padding:1em; border-radius:6px; overflow:auto;">
<code id="r-packages-install" style="font-family: monospace;">
&#35; This installs (if needed) and loads CRAN, Bioconductor, and GitHub packages.
&#35; Run this block once or whenever packages need to be installed/updated.

&#35; A) CRAN packages
cran_packages &lt;- c(
  "readxl",
  "dplyr",
  "tibble",
  "ggplot2",
  "RColorBrewer",
  "paletteer",
  "file2meco",
  "microeco",
  "GUniFrac",    &#35; for UniFrac distances
  "vegan"        &#35; often needed for ordination or diversity calculations
)

missing_cran &lt;- cran_packages[!(cran_packages %in% installed.packages()[,"Package"])]
if(length(missing_cran)) install.packages(missing_cran)

&#35; B) Bioconductor packages (phyloseq, etc.)
if (!requireNamespace("BiocManager", quietly = TRUE))
  install.packages("BiocManager")

if (!"phyloseq" %in% installed.packages()[,"Package"]) {
  BiocManager::install("phyloseq")
}

&#35; C) GitHub package for qiime2R
if (!"qiime2R" %in% installed.packages()[,"Package"]) {
  if (!requireNamespace("devtools", quietly = TRUE))
    install.packages("devtools")
  devtools::install_github("jbisanz/qiime2R")
}

&#35; D) (Optional) microbiome package for taxa_filter()
&#35; if (!"microbiome" %in% installed.packages()[,"Package"]) {
&#35;   install.packages("microbiome")
&#35; }

&#35; Now load libraries
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
&#35; library(vegan)       &#35; If you need direct vegan functions
&#35; library(microbiome)  &#35; If you need taxa_filter()
</code>
  </pre>
  <button onclick="copyCode('r-packages-install')" style="
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
## 3️⃣ Environment Setup
<div style="position: relative; margin-bottom: 1em;">
  <pre style="background:#f6f8fa; padding:1em; border-radius:6px; overflow:auto;">
<code id="r-env-setup" style="font-family: monospace;">
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

## 4️⃣ Import Metadata and Create a Phyloseq Object

You’ll read your metadata from Excel using `readxl`, and convert Qiime2 artifacts using `qza_to_phyloseq()`. Then merge all into a `phyloseq` object.

<div style="position: relative; margin-bottom: 1em;">
  <pre style="background:#f6f8fa; padding:1em; border-radius:6px; overflow:auto;">
<code id="r-phyloseq-obj" style="font-family: monospace;">
&#35; 4.1 Read in sample metadata from an Excel file
samples_df &lt;- read_excel(METADATA_EXCEL, sheet = METADATA_SHEET)

&#35; 4.2 Convert the "sample" column to row names
samples_df &lt;- samples_df %&gt;%
  tibble::column_to_rownames("sample")

&#35; 4.3 Convert Qiime2 artifacts into phyloseq-compatible objects
otu_mat &lt;- qza_to_phyloseq(features = FEATURE_TABLE_QZA)
tax_mat &lt;- qza_to_phyloseq(taxonomy = TAXONOMY_QZA)
tree    &lt;- qza_to_phyloseq(tree = TREE_QZA)

&#35; 4.4 Create a sample_data object from the metadata
samples &lt;- sample_data(samples_df, errorIfNULL = FALSE)

&#35; 4.5 Merge all components into a single phyloseq object
pseq &lt;- phyloseq(otu_mat, tax_mat, samples, tree)

&#35; Print info about the new phyloseq object
pseq
</code>
  </pre>
  <button onclick="copyCode('r-phyloseq-obj')" style="
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

## 5️⃣ Filter Taxa

Remove:
- Unassigned or ambiguous taxa
- Mitochondria, Chloroplasts
- Eukaryota

Optional filtering by minimum frequency is commented.

<div style="position: relative; margin-bottom: 1em;">
  <pre style="background:#f6f8fa; padding:1em; border-radius:6px; overflow:auto;">
<code id="r-taxa-filter" style="font-family: monospace;">
&#35; Customize these filters as necessary for your dataset

&#35; 5.1 Remove unknown or unassigned phyla
pseq &lt;- subset_taxa(pseq, !is.na(Phylum) &amp; !Phylum %in% c("", "Unassigned"))

&#35; 5.2 Remove Eukaryota (Kingdom)
pseq &lt;- subset_taxa(pseq, !is.na(Kingdom) &amp; !Kingdom %in% c("", "Eukaryota"))

&#35; 5.3 Remove Unassigned (Kingdom)
pseq &lt;- subset_taxa(pseq, !is.na(Kingdom) &amp; !Kingdom %in% c("", "Unassigned"))

&#35; 5.4 Remove Cyanobacteria (Phylum)
pseq &lt;- subset_taxa(pseq, !is.na(Phylum) &amp; !Phylum %in% c("", "Cyanobacteria"))

&#35; 5.5 Remove Mitochondria (Family)
pseq &lt;- subset_taxa(pseq, !is.na(Family) &amp; !Family %in% c("", "Mitochondria"))

&#35; 5.6 Remove Chloroplast (Class)
pseq &lt;- subset_taxa(pseq, !is.na(Class) &amp; !Class %in% c("", "Chloroplast"))

&#35; (Optional) Filter OTUs by minimum frequency threshold
&#35; pseq &lt;- taxa_filter(pseq, frequency = 0.001, below = FALSE, drop_samples = TRUE)

&#35; Summaries
ntaxa(pseq)
nsamples(pseq)
sample_names(pseq)[1:5]
rank_names(pseq)
sample_variables(pseq)
otu_table(pseq)[1:5, 1:5]
tax_table(pseq)[1:5, 1:4]
phy_tree(pseq)
taxa_names(pseq)[1:10]

&#35; Prune to top 10 most abundant taxa (example)
myTaxa &lt;- names(sort(taxa_sums(pseq), decreasing = TRUE)[1:10])
ex1    &lt;- prune_taxa(myTaxa, pseq)

&#35; Quick checks/plots
plot(phy_tree(ex1), show.node.label = TRUE)
plot_tree(ex1, color = TREATMENT_VAR, label.tips = "Phylum",
          ladderize = "left", justify = "left", size = "Abundance")

&#35; Save the processed phyloseq object for future use
save.image(file = PSEQ_RDATA)
</code>
  </pre>
  <button onclick="copyCode('r-taxa-filter')" style="
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

