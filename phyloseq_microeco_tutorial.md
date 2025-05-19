---
layout: default
title: 16S Phyloseq & Microeco Analysis Tutorial
---
<script>
  document.documentElement.style.visibility = 'hidden';

  var correctPassword = "cqubioinfo2025";
  var userInput = prompt("🔒 Enter password to view this page, If you don't have password please contact y.sharmabajagai@cqu.edu.au:");

  if (userInput === correctPassword) {
    document.documentElement.style.visibility = 'visible';
  } else {
    document.write(`
      <div style="text-align:center; padding-top:50px; font-family:sans-serif;">
        <h2 style="color:#c00;">🚫 Access Denied</h2>
        <p style="font-size:18px;">This content is currently restricted to enrolled students.</p>
        <p style="font-size:16px;">
          If you would like access to these training materials, please contact:<br>
          <a href="mailto:y.sharmabajagai@cqu.edu.au">y.sharmabajagai@cqu.edu.au</a>
        </p>
      </div>
    `);
  }
</script>


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

## 6️⃣ Import phyloseq object to Microeco and prepare for analysis

- Convert `phyloseq` → `microeco`
- Rarefy to `RAREFACTION_DEPTH`
- Compute and save:
  - Taxonomic abundance
  - Alpha & beta diversity (UniFrac, PD, Chao1, etc.)
  - Barplots by group or sample

Plots saved as both `.pdf` and `.png`.
<div style="position: relative; margin-bottom: 1em;">
  <pre style="background:#f6f8fa; padding:1em; border-radius:6px; overflow:auto;">
<code id="r-microeco-setup" style="font-family: monospace;">
&#35; Create directory for microeco outputs
dir.create(MECO_DIR, showWarnings = FALSE)

&#35; copy the file `PSEQ_RDATA` into the microeco directory,
&#35; under the same file name.
file.copy(
  from      = PSEQ_RDATA,
  to        = file.path(MECO_DIR, basename(PSEQ_RDATA)), 
  overwrite = TRUE
)

&#35; Switch to microeco folder
setwd(MECO_DIR)  

&#35; load pseq data
load(PSEQ_RDATA) 

&#35; Convert phyloseq object to microeco dataset
dataset &lt;- phyloseq2meco(pseq)

&#35; Summaries
dataset$sample_sums() %&gt;% range
dataset$sample_sums()

&#35; go the beginning of the script and set RAREFACTION_DEPTH

&#35; Rarefy to a uniform depth
dataset$rarefy_samples(sample.size = RAREFACTION_DEPTH)

&#35; Re-check sample sums after rarefaction
dataset$sample_sums() %&gt;% range

&#35; Save basic files
dataset$save_table(dirpath = "basic_files", sep = ",")

&#35; Calculate &amp; save abundance at each taxonomic rank
dataset$cal_abund()
dataset$save_abund(dirpath = "taxa_abund")

&#35; Alpha diversity (including PD), then save
dataset$cal_alphadiv(PD = TRUE)
dataset$save_alphadiv(dirpath = "alpha_diversity")

&#35; Beta diversity (including UniFrac), then save
dataset$cal_betadiv(unifrac = TRUE)
dataset$save_betadiv(dirpath = "beta_diversity")
</code>
  </pre>
  <button onclick="copyCode('r-microeco-setup')" style="
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
## 7️⃣ Bar plots for relative abundacne

<div style="position: relative; margin-bottom: 1em;">
  <pre style="background:#f6f8fa; padding:1em; border-radius:6px; overflow:auto; max-height:800px;">
<code id="barplot-taxonomic-ranks" style="font-family: monospace;"># We'll create per-sample (faceted by TREATMENT_VAR) and group-mean bar plots
# for major taxonomic ranks: Phylum, Class, Order, Family, Genus, etc.

#We'll make both (a) per-sample bar plots (faceted by Treatment)
#and (b) group-mean bar plots, for each major taxonomic rank.

#################################
#7.1 PHYLUM
#################################

#7.1.1 Top 10 phyla (per-sample bar plot, faceted by Treatment)
t1_Phylum <- trans_abund$new(dataset = dataset, taxrank = "Phylum", ntaxa = 10)

bar_plot_Phylum_Treatment <- t1_Phylum$plot_bar(
  color_values = paletteer::paletteer_d("ggthemes::Classic_20"),
  bar_full     = TRUE,
  others_color = "grey90",
  facet        = TREATMENT_VAR,
  strip_text   = 18,
  legend_text_italic = FALSE,
  xtext_angle  = 60,
  xtext_size   = 16,
  xtext_keep   = TRUE,
  xtitle_keep  = TRUE,
  ytitle_size  = 17
)

bar_plot_Phylum_Treatment <- bar_plot_Phylum_Treatment +
  theme(
    legend.title = element_text(size = 18, face = "bold"), 
    legend.text  = element_text(size = 16), 
    axis.text.y  = element_text(size = 18),
    axis.title.y = element_text(size = 20, face = "bold")
  )

ggsave("bar_plot_Phylum_Treatment.pdf", plot = bar_plot_Phylum_Treatment, device = "pdf")
ggsave("bar_plot_Phylum_Treatment.png", plot = bar_plot_Phylum_Treatment, device = "png")

#################################
#7.2 CLASS
#################################

#7.2.1 Top 20 classes (per-sample bar plot, faceted by Treatment)
t1_Class <- trans_abund$new(dataset = dataset, taxrank = "Class", ntaxa = 20)

bar_plot_Class_Treatment <- t1_Class$plot_bar(
  color_values = paletteer::paletteer_d("ggthemes::Classic_20"),
  bar_full     = TRUE,
  others_color = "grey90",
  facet        = TREATMENT_VAR,
  strip_text   = 18,
  legend_text_italic = FALSE,
  xtext_angle  = 60,
  xtext_size   = 16
)

bar_plot_Class_Treatment <- bar_plot_Class_Treatment +
  theme(
    legend.title = element_text(size = 18, face = "bold"), 
    legend.text  = element_text(size = 16), 
    axis.text.y  = element_text(size = 18),
    axis.title.y = element_text(size = 20, face = "bold")
  )

ggsave("bar_plot_Class_Treatment.pdf", 
       plot = bar_plot_Class_Treatment, 
       device = "pdf", width = 9, height = 6)

ggsave("bar_plot_Class_Treatment.png", 
       plot = bar_plot_Class_Treatment, 
       device = "png", width = 9, height = 6)

#7.2.2 Class group-mean bar plot by Treatment
t1_Class_group_Treatment <- trans_abund$new(
  dataset   = dataset,
  taxrank   = "Class",
  ntaxa     = 20,
  groupmean = TREATMENT_VAR
)

bar_plot_Class_group_Treatment <- t1_Class_group_Treatment$plot_bar(
  color_values = paletteer::paletteer_d("ggthemes::Classic_20"),
  bar_full     = TRUE,
  others_color = "grey90",
  strip_text   = 18,
  legend_text_italic = FALSE,
  xtext_angle  = 60,
  xtext_size   = 16
)

bar_plot_Class_group_Treatment <- bar_plot_Class_group_Treatment +
  theme(
    legend.title = element_text(size = 18, face = "bold"), 
    legend.text  = element_text(size = 16), 
    axis.text.y  = element_text(size = 18),
    axis.text.x  = element_text(size = 20, face = "bold"),
    axis.title.y = element_text(size = 20, face = "bold")
  )

ggsave("bar_plot_Class_group_Treatment.pdf", 
       plot = bar_plot_Class_group_Treatment, 
       device = "pdf")

ggsave("bar_plot_Class_group_Treatment.png", 
       plot = bar_plot_Class_group_Treatment, 
       device = "png")

#################################
#7.3 ORDER
#################################

#7.3.1 Top 20 orders (per-sample bar plot, faceted by Treatment)
t1_Order <- trans_abund$new(dataset = dataset, taxrank = "Order", ntaxa = 20)

bar_plot_Order_Treatment <- t1_Order$plot_bar(
  color_values = paletteer::paletteer_d("ggthemes::Classic_20"),
  bar_full     = TRUE,
  others_color = "grey90",
  facet        = TREATMENT_VAR,
  strip_text   = 18,
  legend_text_italic = FALSE,
  xtext_angle  = 60,
  xtext_size   = 16
)

bar_plot_Order_Treatment <- bar_plot_Order_Treatment +
  theme(
    legend.title = element_text(size = 18, face = "bold"), 
    legend.text  = element_text(size = 16), 
    axis.text.y  = element_text(size = 18), 
    axis.title.y = element_text(size = 20, face = "bold")
  )

ggsave("bar_plot_Order_Treatment.pdf", 
       plot = bar_plot_Order_Treatment, 
       device = "pdf", width = 9, height = 6)

ggsave("bar_plot_Order_Treatment.png", 
       plot = bar_plot_Order_Treatment, 
       device = "png", width = 9, height = 6)
#7.3.2 Order group-mean bar plot by Treatment
t1_Order_group_Treatment <- trans_abund$new(
  dataset   = dataset,
  taxrank   = "Order",
  ntaxa     = 20,
  groupmean = TREATMENT_VAR
)
bar_plot_Order_group_Treatment <- t1_Order_group_Treatment$plot_bar(
  color_values = paletteer::paletteer_d("ggthemes::Classic_20"),
  bar_full     = TRUE,
  others_color = "grey90",
  strip_text   = 18,
  legend_text_italic = FALSE,
  xtext_angle  = 60,
  xtext_size   = 16
)
bar_plot_Order_group_Treatment <- bar_plot_Order_group_Treatment +
  theme(
    legend.title = element_text(size = 18, face = "bold"), 
    legend.text  = element_text(size = 16), 
    axis.text.y  = element_text(size = 18),
    axis.text.x  = element_text(size = 20, face = "bold"),
    axis.title.y = element_text(size = 20, face = "bold")
  )

ggsave("bar_plot_Order_group_Treatment.pdf", 
       plot = bar_plot_Order_group_Treatment, 
       device = "pdf")

ggsave("bar_plot_Order_group_Treatment.png", 
       plot = bar_plot_Order_group_Treatment, 
       device = "png")

#################################
#7.4 FAMILY
#################################

#7.4.1 Top 20 families (per-sample bar plot, faceted by Treatment)
t1_Family <- trans_abund$new(dataset = dataset, taxrank = "Family", ntaxa = 20)

bar_plot_Family_Treatment <- t1_Family$plot_bar(
  color_values = paletteer::paletteer_d("ggthemes::Classic_20"),
  bar_full     = TRUE,
  others_color = "grey90",
  facet        = TREATMENT_VAR,
  strip_text   = 18,
  legend_text_italic = FALSE,
  xtext_angle  = 60,
  xtext_size   = 16
)

bar_plot_Family_Treatment <- bar_plot_Family_Treatment +
  theme(
    legend.title = element_text(size = 18, face = "bold"), 
    legend.text  = element_text(size = 16), 
    axis.text.y  = element_text(size = 18), 
    axis.title.y = element_text(size = 20, face = "bold")
  )

ggsave("bar_plot_Family_Treatment.pdf", 
       plot = bar_plot_Family_Treatment, 
       device = "pdf", width = 9, height = 6)

ggsave("bar_plot_Family_Treatment.png", 
       plot = bar_plot_Family_Treatment, 
       device = "png", width = 9, height = 6)

#7.4.2 Family group-mean bar plot by Treatment
t1_Family_group_Treatment <- trans_abund$new(
  dataset   = dataset,
  taxrank   = "Family",
  ntaxa     = 20,
  groupmean = TREATMENT_VAR
)

bar_plot_Family_group_Treatment <- t1_Family_group_Treatment$plot_bar(
  color_values = paletteer::paletteer_d("ggthemes::Classic_20"),
  bar_full     = TRUE,
  others_color = "grey90",
  strip_text   = 18,
  legend_text_italic = FALSE,
  xtext_angle  = 60,
  xtext_size   = 16
)

bar_plot_Family_group_Treatment <- bar_plot_Family_group_Treatment +
  theme(
    legend.title = element_text(size = 18, face = "bold"), 
    legend.text  = element_text(size = 16), 
    axis.text.y  = element_text(size = 18),
    axis.text.x  = element_text(size = 20, face = "bold"),
    axis.title.y = element_text(size = 20, face = "bold")
  )

ggsave("bar_plot_Family_group_Treatment.pdf", 
       plot = bar_plot_Family_group_Treatment, 
       device = "pdf")

ggsave("bar_plot_Family_group_Treatment.png", 
       plot = bar_plot_Family_group_Treatment, 
       device = "png")

#################################
#7.5 GENUS
#################################

#7.7.1 Top 20 genera (per-sample bar plot, faceted by Treatment)
t1_Genus <- trans_abund$new(dataset = dataset, taxrank = "Genus", ntaxa = 20)
bar_plot_Genus_Treatment <- t1_Genus$plot_bar(
  color_values = paletteer::paletteer_d("ggthemes::Classic_20"),
  bar_full     = TRUE,
  others_color = "grey90",
  facet        = TREATMENT_VAR,
  strip_text   = 18,
  legend_text_italic = TRUE,
  xtext_angle  = 60,
  xtext_size   = 16
)
bar_plot_Genus_Treatment <- bar_plot_Genus_Treatment +
  theme(
    legend.title = element_text(size = 18, face = "bold"), 
    legend.text  = element_text(size = 16), 
    axis.text.y  = element_text(size = 18), 
    axis.title.y = element_text(size = 20, face = "bold")
  )
ggsave("bar_plot_Genus_Treatment.pdf", 
       plot = bar_plot_Genus_Treatment, 
       device = "pdf", width = 9, height = 6)

ggsave("bar_plot_Genus_Treatment.png", 
       plot = bar_plot_Genus_Treatment, 
       device = "png", width = 9, height = 6)
#7.7.2 Genus group-mean bar plot by Treatment
t1_Genus_group_Treatment <- trans_abund$new(
  dataset   = dataset,
  taxrank   = "Genus",
  ntaxa     = 20,
  groupmean = TREATMENT_VAR
)
bar_plot_Genus_group_Treatment <- t1_Genus_group_Treatment$plot_bar(
  color_values = paletteer::paletteer_d("ggthemes::Classic_20"),
  bar_full     = TRUE,
  others_color = "grey90",
  strip_text   = 18,
  legend_text_italic = TRUE,
  xtext_angle  = 60,
  xtext_size   = 16
)
bar_plot_Genus_group_Treatment <- bar_plot_Genus_group_Treatment +
  theme(
    legend.title = element_text(size = 18, face = "bold"), 
    legend.text  = element_text(size = 16), 
    axis.text.y  = element_text(size = 18),
    axis.text.x  = element_text(size = 20, face = "bold"),
    axis.title.y = element_text(size = 20, face = "bold")
  )

ggsave("bar_plot_Genus_group_Treatment.pdf", 
       plot = bar_plot_Genus_group_Treatment, 
       device = "pdf")

ggsave("bar_plot_Genus_group_Treatment.png", 
       plot = bar_plot_Genus_group_Treatment, 
       device = "png")
</code>
  </pre>
  <button onclick="copyCode('barplot-taxonomic-ranks')" style="
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

## 8️⃣ Alpha & Beta Diversity

- `trans_alpha$new()` calculates richness metrics like Shannon, Simpson
- `trans_beta$new()` computes ordinations like PCoA, NMDS with UniFrac

Boxplots and ordination plots are saved in your output directory.

<div style="position: relative; margin-bottom: 1em;">
  <pre style="background:#f6f8fa; padding:1em; border-radius:6px; overflow:auto; max-height:800px;">
<code id="alpha-and-beta-diversity" style="font-family: monospace;">
###############################################################################
#ALPHA DIVERSITY
###############################################################################
#Example: group by TREATMENT_VAR
t1_alpha <- trans_alpha$new(dataset = dataset, group = TREATMENT_VAR)

#Alpha diversity summary
alpha_summary <- t1_alpha$data_stat
write.csv(alpha_summary, "alpha_summary.csv")

#Conduct group-difference tests (Kruskal-Wallis, etc.)
t1_alpha$cal_diff(method = "KW")
alpha_KW <- t1_alpha$res_diff
write.csv(alpha_KW, "alpha_diff_KW.csv")

#Plot alpha diversity metrics
my_color_palette <- brewer.pal(n = 9, name = "Set1")

metrics_to_plot <- c("Shannon", "Simpson", "Observed", "InvSimpson", "Chao1")
for (m in metrics_to_plot) {
  plot_obj <- t1_alpha$plot_alpha(
    measure      = m,
    color_values = my_color_palette,
    xtext_size   = 15,
    ytext_size   = 15,
    add_sig      = TRUE
  ) + theme(axis.text.x = element_text(angle = 90, vjust = 0.5, hjust = 1))
  
  pdf_file <- paste0(m, "_", TREATMENT_VAR, ".pdf")
  pdf(pdf_file)
  print(plot_obj)
  dev.off()
}

###############################################################################
#BETA DIVERSITY (ORDINATION)
###############################################################################
#Weighted UniFrac
t1_wei <- trans_beta$new(dataset = dataset, group = TREATMENT_VAR, measure = "wei_unifrac")

#Weighted UniFrac: PCoA
t1_wei$cal_ordination(method = "PCoA")
pcoa_wei <- t1_wei$plot_ordination(
  plot_color             = TREATMENT_VAR,
  color_values           = my_color_palette,
  plot_type              = "point",
  centroid_segment_alpha = 1,
  point_alpha            = 1,
  centroid_segment_size  = 0.5,
  centroid_segment_linetype = 3
) + theme_classic() +
  geom_vline(xintercept = 0, linetype = 2) +
  geom_hline(yintercept = 0, linetype = 2)
ggsave("pcoa_wei_unifrac.pdf", plot = pcoa_wei, width = 6, height = 5)

#Weighted UniFrac: NMDS
t1_wei$cal_ordination(method = "NMDS")
nmds_wei <- t1_wei$plot_ordination(
  plot_color             = TREATMENT_VAR,
  color_values           = my_color_palette,
  plot_type              = "point",
  centroid_segment_alpha = 1,
  point_alpha            = 1,
  centroid_segment_size  = 0.5,
  centroid_segment_linetype = 3
) + theme_classic() +
  geom_vline(xintercept = 0, linetype = 2) +
  geom_hline(yintercept = 0, linetype = 2)
ggsave("nmds_wei_unifrac.pdf", plot = nmds_wei, width = 6, height = 5)

#Unweighted UniFrac
t1_unwei <- trans_beta$new(dataset = dataset, group = TREATMENT_VAR, measure = "unwei_unifrac")

#Unweighted UniFrac: PCoA
t1_unwei$cal_ordination(method = "PCoA")
pcoa_unwei <- t1_unwei$plot_ordination(
  plot_color             = TREATMENT_VAR,
  color_values           = my_color_palette,
  plot_type              = "point",
  centroid_segment_alpha = 1,
  point_alpha            = 1
) + theme_classic() +
  geom_vline(xintercept = 0, linetype = 2) +
  geom_hline(yintercept = 0, linetype = 2)
ggsave("pcoa_unwei_unifrac.pdf", plot = pcoa_unwei, width = 6, height = 5)

#Unweighted UniFrac: NMDS
t1_unwei$cal_ordination(method = "NMDS")
nmds_unwei <- t1_unwei$plot_ordination(
  plot_color             = TREATMENT_VAR,
  color_values           = my_color_palette,
  plot_type              = "point",
  centroid_segment_alpha = 1,
  point_alpha            = 1
) + theme_classic() +
  geom_vline(xintercept = 0, linetype = 2) +
  geom_hline(yintercept = 0, linetype = 2)
ggsave("nmds_unwei_unifrac.pdf", plot = nmds_unwei, width = 6, height = 5)
</code>
  </pre>
  <button onclick="copyCode('barplot-taxonomic-ranks')" style="
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
## 9️⃣Group distance and PERMANOVA

<div style="position: relative; margin-bottom: 1em;">
  <pre style="background:#f6f8fa; padding:1em; border-radius:6px; overflow:auto; max-height:800px;">
<code id="group-distance-and-permanova" style="font-family: monospace;">

##Note: change the method based on the number of groups / required test method
#Weighted UniFrac
t1_wei$cal_group_distance(within_group = TRUE)
t1_wei$cal_group_distance_diff(method = "KW")
write.csv(t1_wei$res_group_distance_diff, "wei_unifrac_distance_diff.csv")

dist_plot_wei <- t1_wei$plot_group_distance(
  boxplot_add = "mean",
  color_values = my_color_palette
) + theme(axis.text.x = element_text(angle = 90, vjust = 0.5, hjust = 1))
ggsave("wei_unifrac_distance_boxplot.pdf", plot = dist_plot_wei)

t1_wei$cal_manova(manova_all = FALSE)
write.csv(t1_wei$res_manova, "permanova_wei_unifrac.csv")

#Unweighted UniFrac
t1_unwei$cal_group_distance(within_group = TRUE)
t1_unwei$cal_group_distance_diff(method = "KW")
write.csv(t1_unwei$res_group_distance_diff, "unwei_unifrac_distance_diff.csv")

dist_plot_unwei <- t1_unwei$plot_group_distance(
  boxplot_add = "mean",
  color_values = my_color_palette
) + theme(axis.text.x = element_text(angle = 90, vjust = 0.5, hjust = 1))
ggsave("unwei_unifrac_distance_boxplot.pdf", plot = dist_plot_unwei)

t1_unwei$cal_manova(manova_all = FALSE)
write.csv(t1_unwei$res_manova, "permanova_unwei_unifrac.csv")
</code>
  </pre>
  <button onclick="copyCode('barplot-taxonomic-ranks')" style="
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
## 🔟 Differential Abundance (LEfSe & Metastat)

- LEfSe for multiple taxonomic levels using `trans_diff$new(...)`
- Metastat also included as an alternate method
- Results exported as CSV files
- Try-catch blocks ensure the script continues even if some tax ranks fail

<div style="position: relative; margin-bottom: 1em;">
  <pre style="background:#f6f8fa; padding:1em; border-radius:6px; overflow:auto; max-height:800px;">
<code id="differential-taxa" style="font-family: monospace;">

###############################################################################
#LEfSe
###############################################################################

#LEfSe at multiple tax ranks
#The "group" is TREATMENT_VAR
#Adjust filter_thres, alpha, p_adjust_method, etc. as needed.

####10.1.1 All levels
tryCatch({
  t1_lefse_all <- trans_diff$new(
    dataset        = dataset,
    method         = "lefse",
    group          = TREATMENT_VAR,
    taxa_level     = "all",
    filter_thres   = 0,
    alpha          = 0.05,
    p_adjust_method= "none",
    lefse_subgroup = NULL
  )
  lefse_all <- t1_lefse_all$plot_diff_bar(keep_prefix = FALSE)
  pdf("lefse_all.pdf")
  print(lefse_all)
  dev.off()
}, error = function(e) {
  message("Error in LEfSe (all levels): ", e$message)
  #You could optionally: dev.off() if it partially opened a pdf
})

####10.1.2 Phylum
tryCatch({
  t1_lefse_Phylum <- trans_diff$new(
    dataset        = dataset,
    method         = "lefse",
    group          = TREATMENT_VAR,
    taxa_level     = "Phylum",
    filter_thres   = 0,
    alpha          = 0.05,
    p_adjust_method= "none"
  )
  lefse_Phylum <- t1_lefse_Phylum$plot_diff_bar(keep_prefix = FALSE)
  pdf("lefse_Phylum.pdf")
  print(lefse_Phylum)
  dev.off()
}, error = function(e) {
  message("Error in LEfSe (Phylum): ", e$message)
})

####10.1.3 Class
tryCatch({
  t1_lefse_Class <- trans_diff$new(
    dataset        = dataset,
    method         = "lefse",
    group          = TREATMENT_VAR,
    taxa_level     = "Class",
    filter_thres   = 0,
    alpha          = 0.05,
    p_adjust_method= "none"
  )
  lefse_Class <- t1_lefse_Class$plot_diff_bar(keep_prefix = FALSE)
  pdf("lefse_Class.pdf")
  print(lefse_Class)
  dev.off()
}, error = function(e) {
  message("Error in LEfSe (Class): ", e$message)
})

####10.1.4 Order
tryCatch({
  t1_lefse_Order <- trans_diff$new(
    dataset        = dataset,
    method         = "lefse",
    group          = TREATMENT_VAR,
    taxa_level     = "Order",
    filter_thres   = 0,
    alpha          = 0.05,
    p_adjust_method= "none"
  )
  lefse_Order <- t1_lefse_Order$plot_diff_bar(keep_prefix = FALSE)
  pdf("lefse_Order.pdf")
  print(lefse_Order)
  dev.off()
}, error = function(e) {
  message("Error in LEfSe (Order): ", e$message)
})

####10.1.5 Family
tryCatch({
  t1_lefse_Family <- trans_diff$new(
    dataset        = dataset,
    method         = "lefse",
    group          = TREATMENT_VAR,
    taxa_level     = "Family",
    filter_thres   = 0,
    alpha          = 0.05,
    p_adjust_method= "none"
  )
  lefse_Family <- t1_lefse_Family$plot_diff_bar(keep_prefix = FALSE)
  pdf("lefse_Family.pdf")
  print(lefse_Family)
  dev.off()
}, error = function(e) {
  message("Error in LEfSe (Family): ", e$message)
})

####10.1.6 Genus
tryCatch({
  t1_lefse_Genus <- trans_diff$new(
    dataset        = dataset,
    method         = "lefse",
    group          = TREATMENT_VAR,
    taxa_level     = "Genus",
    filter_thres   = 0,
    alpha          = 0.05,
    p_adjust_method= "none"
  )
  lefse_Genus <- t1_lefse_Genus$plot_diff_bar(keep_prefix = FALSE)
  pdf("lefse_Genus.pdf")
  print(lefse_Genus)
  dev.off()
}, error = function(e) {
  message("Error in LEfSe (Genus): ", e$message)
})

###############################################################################
#Metastat analysis
#Another approach for differential abundance at different taxonomic levels
###############################################################################

#Genus level
t1_Treatment_metastat <- trans_diff$new(
  dataset    = dataset,
  method     = "metastat",
  group      = TREATMENT_VAR,
  taxa_level = "Genus"
)

metastat_diff_result_Treatment <- t1_Treatment_metastat$res_diff
write.csv(metastat_diff_result_Treatment, "metastat_diff_result_Treatment.csv")

metastat_group_abund_Treatment <- t1_Treatment_metastat$res_abund
write.csv(metastat_group_abund_Treatment, "metastat_group_abund_Treatment.csv")
#Repeat for Phylum
t1_Treatment_metastat_Phylum <- trans_diff$new(
  dataset    = dataset,
  method     = "metastat",
  group      = TREATMENT_VAR,
  taxa_level = "Phylum"
)
metastat_diff_result_Treatment_Phylum <- t1_Treatment_metastat_Phylum$res_diff
write.csv(metastat_diff_result_Treatment_Phylum, "metastat_diff_result_Treatment_Phylum.csv")

metastat_group_abund_Treatment_Phylum <- t1_Treatment_metastat_Phylum$res_abund
write.csv(metastat_group_abund_Treatment_Phylum, "metastat_group_abund_Treatment_Phylum.csv")

#Class

t1_Treatment_metastat_Phulym <- trans_diff$new(dataset = dataset, method = "metastat", group = TREATMENT_VAR, taxa_level = "Class")

metastat_diff_result_Treatment_Class <- t1_Treatment_metastat_Phulym$res_diff

write.csv(metastat_diff_result_Treatment_Class, "metastat_diff_result_Treatment_Class.csv")

metastat_group_abund_Treatment_Class <- t1_Treatment_metastat_Phulym$res_abund

write.csv(metastat_group_abund_Treatment_Class, "metastat_group_abund_Treatment_Class.csv")

#Family

t1_Treatment_metastat_Phulym <- trans_diff$new(dataset = dataset, method = "metastat", group = TREATMENT_VAR, taxa_level = "Family")

metastat_diff_result_Treatment_Family <- t1_Treatment_metastat_Phulym$res_diff

write.csv(metastat_diff_result_Treatment_Family, "metastat_diff_result_Treatment_Family.csv")

metastat_group_abund_Treatment_Family <- t1_Treatment_metastat_Phulym$res_abund

write.csv(metastat_group_abund_Treatment_Family, "metastat_group_abund_Treatment_Family.csv")

#Order
t1_Treatment_metastat_Phulym <- trans_diff$new(dataset = dataset, method = "metastat", group = TREATMENT_VAR, taxa_level = "Order")

metastat_diff_result_Treatment_Order <- t1_Treatment_metastat_Phulym$res_diff

write.csv(metastat_diff_result_Treatment_Order, "metastat_diff_result_Treatment_Order.csv")

metastat_group_abund_Treatment_Order <- t1_Treatment_metastat_Phulym$res_abund

write.csv(metastat_group_abund_Treatment_Order, "metastat_group_abund_Treatment_Order.csv")
</code>
  </pre>
  <button onclick="copyCode('barplot-taxonomic-ranks')" style="
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
##  Final Save
<div style="position: relative; margin-bottom: 1em;">
  <pre style="background:#f6f8fa; padding:1em; border-radius:6px; overflow:auto; max-height:800px;">
<code id="final-save" style="font-family: monospace;">
writeLines(capture.output(sessionInfo()), SESSION_INFO)
save.image(file = FINAL_WORKSPACE)

message("All analyses complete. Session info and workspace saved.")

</code>
  </pre>
  <button onclick="copyCode('barplot-taxonomic-ranks')" style="
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
