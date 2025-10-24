---
layout: default
title: 16S rRNA - Phyloseq & Microeco Analysis Tutorial
---

<style>
  .codewrap { position: relative; margin: 1rem 0; }
  .codewrap pre {
    background: #f6f8fa; padding: 1em; border-radius: 8px; overflow: auto;
    white-space: pre; /* preserve newlines exactly */
  }
  .copybtn {
    position: absolute; top: 10px; right: 10px;
    background: #0366d6; color: #fff; border: 0; padding: 6px 10px;
    border-radius: 6px; font-size: 0.85em; cursor: pointer;
  }
  h1,h2, h3 { margin-top: 1.2em; }
  .muted { color:#666; }
  .note { background:#fff7cc; border:1px solid #ffe58a; padding:.6em .8em; border-radius:6px; }

  /* Password card */
  .pw-wrap {
    max-width: 460px; margin: 12vh auto 3rem; padding: 1.25rem 1rem;
    border: 1px solid #e5e7eb; border-radius: 10px; background: #fff;
    box-shadow: 0 8px 20px rgba(0,0,0,.05);
    font-family: system-ui, -apple-system, Segoe UI, Roboto, sans-serif;
  }
  .pw-wrap h2 { margin: 0 0 .5rem 0; font-size: 1.35rem; }
  .pw-wrap p { margin: .25rem 0 .75rem; color: #444; }
  .pw-row { display: flex; gap: .5rem; }
  .pw-row input[type="password"] {
    flex: 1; padding: .6rem .7rem; border: 1px solid #cbd5e1; border-radius: 8px;
    font-size: 1rem;
  }
  .pw-row button {
    padding: .6rem .9rem; background:#0366d6; color:#fff; border:0; border-radius:8px;
    font-weight: 600; cursor: pointer;
  }
  .pw-err { color:#c00; margin-top:.5rem; display:none; }
</style>

<!-- Password form (visible by default) -->
<div id="pw-card" class="pw-wrap">
  <h2>🔒 Enter password to access this training page</h2>
  <p>Contact <a href="mailto:y.sharmabajagai@cqu.edu.au">y.sharmabajagai@cqu.edu.au</a> if you need access.</p>
  <form id="pw-form" onsubmit="return false;">
    <div class="pw-row">
      <input id="pw-input" type="password" placeholder="Password" autocomplete="current-password" />
      <button id="pw-submit" type="submit">Unlock</button>
    </div>
    <div id="pw-error" class="pw-err">Incorrect password. Please try again.</div>
  </form>
</div>

<!-- Hidden content container -->
<div id="content" style="display:none;"></div>

{% raw %}
<script>
  // 1) ONE copy function for all code blocks
  function copyCode(id) {
    var el = document.getElementById(id);
    if (!el) return;
    var text = el.innerText;
    navigator.clipboard.writeText(text).then(function () {
      alert("✅ Code copied to clipboard!");
    }).catch(function () {
      alert("⚠️ Copy failed. Please copy manually.");
    });
  }

  // Build the full protected HTML (no <script> tags inside)
  function buildPageHTML() {
    return `
      <h1>16S rRNA Sequencing Data Analysis in R using Phyloseq and Microeco</h1>
      <p>
        This tutorial provides a complete R-based workflow for analyzing 16S rRNA sequencing data using
        <code>qiime2R</code>, <code>phyloseq</code>, and <code>microeco</code>.
      </p>

      <hr />
      <p>
      Before starting R, please copy the example data (demo files) from "/project/2025-sharma-cqu-bioinfo/training/phyloseq_meco/data" to your working directory
      <p>
      <h2>Load R and open RStudio</h2>
      <div class="codewrap">
        <button class="copybtn" onclick="copyCode('bash-load')">📋 Copy</button>
        <pre><code id="bash-load" class="language-bash">module load R-bundle-Bioconductor/3.20-foss-2024a-R-4.4.2
module load rstudio
rstudio</code></pre>
      </div>

      <h2>1️⃣ User-defined Settings</h2>
      <p class="muted">Edit paths/filenames and variables before running.</p>
      <div class="codewrap">
        <button class="copybtn" onclick="copyCode('r-setup')">📋 Copy</button>
        <pre><code id="r-setup">### Seed for reproducibility
MY_SEED <- 2345
### Working directory
WORKDIR <- "XXXXXXX"
### Excel file containing sample metadata
METADATA_EXCEL <- "sample_metadata.xlsx"
METADATA_SHEET <- "sample-metadata"  # The sheet name where metadata is stored
### Names of Qiime2 artifacts to import
FEATURE_TABLE_QZA <- "table.qza"
TAXONOMY_QZA      <- "taxonomy.qza"
TREE_QZA          <- "rooted_tree.qza"
### Optional .RData output file name
PSEQ_RDATA <- "XXX_pseq.RData"
### Directory for microeco results
MECO_DIR <- "./meco"
### Rarefaction depth for microeco
RAREFACTION_DEPTH <-   # you need to come back here later (hint: 2000 for this study)
### For alpha/beta diversity, bar plots, etc.
TREATMENT_VAR <- "Origin.Treatment"   # The name of the variable in your sample_data
### File name to save final workspace & session info
FINAL_WORKSPACE <- "XXXX_final_workspace.RData"
SESSION_INFO    <- "XXXX_sessionInfo.txt"</code></pre>
      </div>

      <h2>2️⃣ Install and Load Required Packages</h2>
      <div class="codewrap">
        <button class="copybtn" onclick="copyCode('r-packages-install')">📋 Copy</button>
        <pre><code id="r-packages-install"># This installs (if needed) and loads CRAN, Bioconductor, and GitHub packages.
# Run this block once or whenever packages need to be installed/updated.

# A) CRAN packages
cran_packages <- c(
  "readxl", "dplyr", "tibble", "ggplot2",
  "RColorBrewer", "paletteer", "file2meco", "microeco",
  "GUniFrac",  # for UniFrac distances
  "vegan"      # often needed for ordination or diversity calculations
)
missing_cran <- cran_packages[!(cran_packages %in% installed.packages()[,"Package"])]
if(length(missing_cran)) install.packages(missing_cran)

# B) Bioconductor packages (phyloseq, etc.)
if (!requireNamespace("BiocManager", quietly = TRUE))
  install.packages("BiocManager")
if (!"phyloseq" %in% installed.packages()[,"Package"]) {
  BiocManager::install("phyloseq")
}

# C) GitHub package for qiime2R
if (!"qiime2R" %in% installed.packages()[,"Package"]) {
  if (!requireNamespace("devtools", quietly = TRUE))
    install.packages("devtools")
  devtools::install_github("jbisanz/qiime2R")
}

# D) Optional microbiome package
# if (!"microbiome" %in% installed.packages()[,"Package"]) {
#   install.packages("microbiome")
# }

# Now load libraries
library(readxl); library(tibble); library(dplyr)
library(qiime2R); library(phyloseq); library(ggplot2)
library(file2meco); library(microeco)
library(RColorBrewer); library(paletteer); library(GUniFrac)
# library(vegan)       # If you need direct vegan functions
# library(microbiome)  # If you need taxa_filter()</code></pre>
      </div>

      <h2>3️⃣ Environment Setup</h2>
      <div class="codewrap">
        <button class="copybtn" onclick="copyCode('r-env-setup')">📋 Copy</button>
        <pre><code id="r-env-setup">set.seed(MY_SEED)        # For reproducibility
rngseed <- MY_SEED       # Keep track of the seed
setwd(WORKDIR)           # Set working directory</code></pre>
      </div>

      <h2>4️⃣ Import Metadata and Create a Phyloseq Object</h2>
      <div class="codewrap">
        <button class="copybtn" onclick="copyCode('r-phyloseq-obj')">📋 Copy</button>
        <pre><code id="r-phyloseq-obj"># 4.1 Read in sample metadata from an Excel file
samples_df <- read_excel(METADATA_EXCEL, sheet = METADATA_SHEET)

# 4.2 Convert the "sample" column to row names
samples_df <- samples_df |>
  tibble::column_to_rownames("sample")

# 4.3 Convert Qiime2 artifacts into phyloseq-compatible objects
otu_mat <- qza_to_phyloseq(features = FEATURE_TABLE_QZA)
tax_mat <- qza_to_phyloseq(taxonomy = TAXONOMY_QZA)
tree    <- qza_to_phyloseq(tree = TREE_QZA)

# 4.4 Create a sample_data object from the metadata
samples <- sample_data(samples_df, errorIfNULL = FALSE)

# 4.5 Merge all components into a single phyloseq object
pseq <- phyloseq(otu_mat, tax_mat, samples, tree)

# Print info about the new phyloseq object
pseq</code></pre>
      </div>

      <h2>5️⃣ Filter Taxa</h2>
      <div class="codewrap">
        <button class="copybtn" onclick="copyCode('r-taxa-filter')">📋 Copy</button>
        <pre><code id="r-taxa-filter"># 5) Customize filters as necessary
pseq <- subset_taxa(pseq, !is.na(Phylum)  & !Phylum  %in% c("", "Unassigned"))
pseq <- subset_taxa(pseq, !is.na(Kingdom) & !Kingdom %in% c("", "Eukaryota"))
pseq <- subset_taxa(pseq, !is.na(Kingdom) & !Kingdom %in% c("", "Unassigned"))
pseq <- subset_taxa(pseq, !is.na(Phylum)  & !Phylum  %in% c("", "Cyanobacteria"))
pseq <- subset_taxa(pseq, !is.na(Family)  & !Family  %in% c("", "Mitochondria"))
pseq <- subset_taxa(pseq, !is.na(Class)   & !Class   %in% c("", "Chloroplast"))

# Optional frequency threshold
# pseq <- taxa_filter(pseq, frequency = 0.001, below = FALSE, drop_samples = TRUE)

# Quick summaries
ntaxa(pseq); nsamples(pseq)
sample_names(pseq)[1:5]
rank_names(pseq)
sample_variables(pseq)
otu_table(pseq)[1:5, 1:5]
tax_table(pseq)[1:5, 1:4]
phy_tree(pseq)
taxa_names(pseq)[1:10]

# Example prune to top 10 abundant taxa
myTaxa <- names(sort(taxa_sums(pseq), decreasing = TRUE)[1:10])
ex1    <- prune_taxa(myTaxa, pseq)
plot(phy_tree(ex1), show.node.label = TRUE)
plot_tree(ex1, color = TREATMENT_VAR, label.tips = "Phylum",
          ladderize = "left", justify = "left", size = "Abundance")

# Save processed object
save.image(file = PSEQ_RDATA)</code></pre>
      </div>

      <h2>6️⃣ Import phyloseq to Microeco</h2>
      <div class="codewrap">
        <button class="copybtn" onclick="copyCode('r-microeco-setup')">📋 Copy</button>
        <pre><code id="r-microeco-setup">dir.create(MECO_DIR, showWarnings = FALSE)
file.copy(from = PSEQ_RDATA, to = file.path(MECO_DIR, basename(PSEQ_RDATA)), overwrite = TRUE)
setwd(MECO_DIR)
load(PSEQ_RDATA)

dataset <- phyloseq2meco(pseq)
dataset$sample_sums() |> range()
dataset$sample_sums()

# Set RAREFACTION_DEPTH at the top before running the next line
dataset$rarefy_samples(sample.size = RAREFACTION_DEPTH)
dataset$sample_sums() |> range()

dataset$save_table(dirpath = "basic_files", sep = ",")
dataset$cal_abund();    dataset$save_abund(dirpath = "taxa_abund")
dataset$cal_alphadiv(PD = TRUE); dataset$save_alphadiv(dirpath = "alpha_diversity")
dataset$cal_betadiv(unifrac = TRUE); dataset$save_betadiv(dirpath = "beta_diversity")</code></pre>
      </div>

      <h2>7️⃣ Bar plots — Phylum</h2>
      <div class="codewrap">
        <button class="copybtn" onclick="copyCode('barplot-phylum')">📋 Copy</button>
        <pre><code id="barplot-phylum"># Top 10 phyla (per-sample, faceted by Treatment)
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
) + theme(
  legend.title = element_text(size = 18, face = "bold"), 
  legend.text  = element_text(size = 16), 
  axis.text.y  = element_text(size = 18),
  axis.title.y = element_text(size = 20, face = "bold")
)
ggsave("bar_plot_Phylum_Treatment.pdf", plot = bar_plot_Phylum_Treatment, device = "pdf")
ggsave("bar_plot_Phylum_Treatment.png", plot = bar_plot_Phylum_Treatment, device = "png")</code></pre>
      </div>
#Phylum - grouped

t1_Phylum_group_Treatment <- trans_abund$new(
  dataset   = dataset,
  taxrank   = "Phylum",
  ntaxa     = 20,
  groupmean = TREATMENT_VAR
)
bar_plot_Phylum_group_Treatment <- t1_Phylum_group_Treatment$plot_bar(
  color_values = paletteer::paletteer_d("ggthemes::Phylumic_20"),
  bar_full     = TRUE,
  others_color = "grey90",
  strip_text   = 18,
  legend_text_italic = FALSE,
  xtext_angle  = 60,
  xtext_size   = 16
) + theme(
  legend.title = element_text(size = 18, face = "bold"), 
  legend.text  = element_text(size = 16), 
  axis.text.y  = element_text(size = 18),
  axis.text.x  = element_text(size = 20, face = "bold"),
  axis.title.y = element_text(size = 20, face = "bold")
)
ggsave("bar_plot_Phylum_group_Treatment.pdf", plot = bar_plot_Phylum_group_Treatment, device = "pdf")
ggsave("bar_plot_Phylum_group_Treatment.png", plot = bar_plot_Phylum_group_Treatment, device = "png")
      <h2>7️⃣ Bar plots — Class (per-sample)</h2>
      <div class="codewrap">
        <button class="copybtn" onclick="copyCode('barplot-class')">📋 Copy</button>
        <pre><code id="barplot-class">t1_Class <- trans_abund$new(dataset = dataset, taxrank = "Class", ntaxa = 20)
bar_plot_Class_Treatment <- t1_Class$plot_bar(
  color_values = paletteer::paletteer_d("ggthemes::Classic_20"),
  bar_full     = TRUE,
  others_color = "grey90",
  facet        = TREATMENT_VAR,
  strip_text   = 18,
  legend_text_italic = FALSE,
  xtext_angle  = 60,
  xtext_size   = 16
) + theme(
  legend.title = element_text(size = 18, face = "bold"), 
  legend.text  = element_text(size = 16), 
  axis.text.y  = element_text(size = 18),
  axis.title.y = element_text(size = 20, face = "bold")
)
ggsave("bar_plot_Class_Treatment.pdf", plot = bar_plot_Class_Treatment, device = "pdf", width = 9, height = 6)
ggsave("bar_plot_Class_Treatment.png", plot = bar_plot_Class_Treatment, device = "png", width = 9, height = 6)</code></pre>
      </div>

      <h2>7️⃣ Bar plots — Class (group mean)</h2>
      <div class="codewrap">
        <button class="copybtn" onclick="copyCode('barplot-class-group')">📋 Copy</button>
        <pre><code id="barplot-class-group">t1_Class_group_Treatment <- trans_abund$new(
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
) + theme(
  legend.title = element_text(size = 18, face = "bold"), 
  legend.text  = element_text(size = 16), 
  axis.text.y  = element_text(size = 18),
  axis.text.x  = element_text(size = 20, face = "bold"),
  axis.title.y = element_text(size = 20, face = "bold")
)
ggsave("bar_plot_Class_group_Treatment.pdf", plot = bar_plot_Class_group_Treatment, device = "pdf")
ggsave("bar_plot_Class_group_Treatment.png", plot = bar_plot_Class_group_Treatment, device = "png")</code></pre>
      </div>

      <h2>7️⃣ Bar plots — Order (per-sample)</h2>
      <div class="codewrap">
        <button class="copybtn" onclick="copyCode('barplot-order')">📋 Copy</button>
        <pre><code id="barplot-order">t1_Order <- trans_abund$new(dataset = dataset, taxrank = "Order", ntaxa = 20)
bar_plot_Order_Treatment <- t1_Order$plot_bar(
  color_values = paletteer::paletteer_d("ggthemes::Classic_20"),
  bar_full     = TRUE,
  others_color = "grey90",
  facet        = TREATMENT_VAR,
  strip_text   = 18,
  legend_text_italic = FALSE,
  xtext_angle  = 60,
  xtext_size   = 16
) + theme(
  legend.title = element_text(size = 18, face = "bold"), 
  legend.text  = element_text(size = 16), 
  axis.text.y  = element_text(size = 18), 
  axis.title.y = element_text(size = 20, face = "bold")
)
ggsave("bar_plot_Order_Treatment.pdf", plot = bar_plot_Order_Treatment, device = "pdf", width = 9, height = 6)
ggsave("bar_plot_Order_Treatment.png", plot = bar_plot_Order_Treatment, device = "png", width = 9, height = 6)</code></pre>
      </div>

      <h2>7️⃣ Bar plots — Order (group mean)</h2>
      <div class="codewrap">
        <button class="copybtn" onclick="copyCode('barplot-order-group')">📋 Copy</button>
        <pre><code id="barplot-order-group">t1_Order_group_Treatment <- trans_abund$new(
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
) + theme(
  legend.title = element_text(size = 18, face = "bold"), 
  legend.text  = element_text(size = 16), 
  axis.text.y  = element_text(size = 18),
  axis.text.x  = element_text(size = 20, face = "bold"),
  axis.title.y = element_text(size = 20, face = "bold")
)
ggsave("bar_plot_Order_group_Treatment.pdf", plot = bar_plot_Order_group_Treatment, device = "pdf")
ggsave("bar_plot_Order_group_Treatment.png", plot = bar_plot_Order_group_Treatment, device = "png")</code></pre>
      </div>

      <h2>7️⃣ Bar plots — Family (per-sample)</h2>
      <div class="codewrap">
        <button class="copybtn" onclick="copyCode('barplot-family')">📋 Copy</button>
        <pre><code id="barplot-family">t1_Family <- trans_abund$new(dataset = dataset, taxrank = "Family", ntaxa = 20)
bar_plot_Family_Treatment <- t1_Family$plot_bar(
  color_values = paletteer::paletteer_d("ggthemes::Classic_20"),
  bar_full     = TRUE,
  others_color = "grey90",
  facet        = TREATMENT_VAR,
  strip_text   = 18,
  legend_text_italic = FALSE,
  xtext_angle  = 60,
  xtext_size   = 16
) + theme(
  legend.title = element_text(size = 18, face = "bold"), 
  legend.text  = element_text(size = 16), 
  axis.text.y  = element_text(size = 18), 
  axis.title.y = element_text(size = 20, face = "bold")
)
ggsave("bar_plot_Family_Treatment.pdf", plot = bar_plot_Family_Treatment, device = "pdf", width = 9, height = 6)
ggsave("bar_plot_Family_Treatment.png", plot = bar_plot_Family_Treatment, device = "png", width = 9, height = 6)</code></pre>
      </div>

      <h2>7️⃣ Bar plots — Family (group mean)</h2>
      <div class="codewrap">
        <button class="copybtn" onclick="copyCode('barplot-family-group')">📋 Copy</button>
        <pre><code id="barplot-family-group">t1_Family_group_Treatment <- trans_abund$new(
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
) + theme(
  legend.title = element_text(size = 18, face = "bold"), 
  legend.text  = element_text(size = 16), 
  axis.text.y  = element_text(size = 18),
  axis.text.x  = element_text(size = 20, face = "bold"),
  axis.title.y = element_text(size = 20, face = "bold")
)
ggsave("bar_plot_Family_group_Treatment.pdf", plot = bar_plot_Family_group_Treatment, device = "pdf")
ggsave("bar_plot_Family_group_Treatment.png", plot = bar_plot_Family_group_Treatment, device = "png")</code></pre>
      </div>

      <h2>7️⃣ Bar plots — Genus (per-sample)</h2>
      <div class="codewrap">
        <button class="copybtn" onclick="copyCode('barplot-genus')">📋 Copy</button>
        <pre><code id="barplot-genus">t1_Genus <- trans_abund$new(dataset = dataset, taxrank = "Genus", ntaxa = 20)
bar_plot_Genus_Treatment <- t1_Genus$plot_bar(
  color_values = paletteer::paletteer_d("ggthemes::Classic_20"),
  bar_full     = TRUE,
  others_color = "grey90",
  facet        = TREATMENT_VAR,
  strip_text   = 18,
  legend_text_italic = TRUE,
  xtext_angle  = 60,
  xtext_size   = 16
) + theme(
  legend.title = element_text(size = 18, face = "bold"), 
  legend.text  = element_text(size = 16), 
  axis.text.y  = element_text(size = 18), 
  axis.title.y = element_text(size = 20, face = "bold")
)
ggsave("bar_plot_Genus_Treatment.pdf", plot = bar_plot_Genus_Treatment, device = "pdf", width = 9, height = 6)
ggsave("bar_plot_Genus_Treatment.png", plot = bar_plot_Genus_Treatment, device = "png", width = 9, height = 6)</code></pre>
      </div>

      <h2>7️⃣ Bar plots — Genus (group mean)</h2>
      <div class="codewrap">
        <button class="copybtn" onclick="copyCode('barplot-genus-group')">📋 Copy</button>
        <pre><code id="barplot-genus-group">t1_Genus_group_Treatment <- trans_abund$new(
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
) + theme(
  legend.title = element_text(size = 18, face = "bold"), 
  legend.text  = element_text(size = 16), 
  axis.text.y  = element_text(size = 18),
  axis.text.x  = element_text(size = 20, face = "bold"),
  axis.title.y = element_text(size = 20, face = "bold")
)
ggsave("bar_plot_Genus_group_Treatment.pdf", plot = bar_plot_Genus_group_Treatment, device = "pdf")
ggsave("bar_plot_Genus_group_Treatment.png", plot = bar_plot_Genus_group_Treatment, device = "png")</code></pre>
      </div>

      <h2>8️⃣ Alpha Diversity</h2>
      <div class="codewrap">
        <button class="copybtn" onclick="copyCode('alpha-diversity')">📋 Copy</button>
        <pre><code id="alpha-diversity"># Alpha diversity
t1_alpha <- trans_alpha$new(dataset = dataset, group = TREATMENT_VAR)

# Alpha diversity summary
alpha_summary <- t1_alpha$data_stat
write.csv(alpha_summary, "alpha_summary.csv")

# Group-difference tests (Kruskal-Wallis)
t1_alpha$cal_diff(method = "KW_dunn") # "KW" =  Non-parametric alternative to one-way ANOVA, # "KW_dunn" = Pairwise comparison for ≥3 groups, "wilcox" = Non-parametric alternative to the two-sample t-test. - choose the right test!
alpha_KW_dunn <- t1_alpha$res_diff
write.csv(alpha_KW_dunn, "alpha_diff_KW_dunn.csv")

# Plot richness and diversity indices
my_color_palette <- RColorBrewer::brewer.pal(n = 9, name = "Set1")
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
  pdf(pdf_file); print(plot_obj); dev.off()
}</code></pre>
      </div>

      <h2>8️⃣ Beta Diversity — Weighted UniFrac (PCoA & NMDS)</h2>
      <div class="codewrap">
        <button class="copybtn" onclick="copyCode('beta-weighted')">📋 Copy</button>
        <pre><code id="beta-weighted">t1_wei <- trans_beta$new(dataset = dataset, group = TREATMENT_VAR, measure = "wei_unifrac")

# PCoA
t1_wei$cal_ordination(method = "PCoA")
pcoa_wei <- t1_wei$plot_ordination(
  plot_color             = TREATMENT_VAR,
  color_values           = my_color_palette,
  plot_type              = c("point", "ellipse"),
  centroid_segment_alpha = 1,
  point_alpha            = 1,
  centroid_segment_size  = 0.5,
  centroid_segment_linetype = 3
) + theme_classic() + geom_vline(xintercept = 0, linetype = 2) + geom_hline(yintercept = 0, linetype = 2)
ggsave("pcoa_wei_unifrac.pdf", plot = pcoa_wei, width = 6, height = 5)

# NMDS
t1_wei$cal_ordination(method = "NMDS")
nmds_wei <- t1_wei$plot_ordination(
  plot_color             = TREATMENT_VAR,
  color_values           = my_color_palette,
  plot_type              = c("point", "ellipse"),
  centroid_segment_alpha = 1,
  point_alpha            = 1,
  centroid_segment_size  = 0.5,
  centroid_segment_linetype = 3
) + theme_classic() + geom_vline(xintercept = 0, linetype = 2) + geom_hline(yintercept = 0, linetype = 2)
ggsave("nmds_wei_unifrac.pdf", plot = nmds_wei, width = 6, height = 5)</code></pre>
      </div>

      <h2>8️⃣ Beta Diversity — Unweighted UniFrac (PCoA & NMDS)</h2>
      <div class="codewrap">
        <button class="copybtn" onclick="copyCode('beta-unweighted')">📋 Copy</button>
        <pre><code id="beta-unweighted">t1_unwei <- trans_beta$new(dataset = dataset, group = TREATMENT_VAR, measure = "unwei_unifrac")

# PCoA
t1_unwei$cal_ordination(method = "PCoA")
pcoa_unwei <- t1_unwei$plot_ordination(
  plot_color             = TREATMENT_VAR,
  color_values           = my_color_palette,
  plot_type              = c("point", "ellipse"),
  centroid_segment_alpha = 1,
  point_alpha            = 1
) + theme_classic() + geom_vline(xintercept = 0, linetype = 2) + geom_hline(yintercept = 0, linetype = 2)
ggsave("pcoa_unwei_unifrac.pdf", plot = pcoa_unwei, width = 6, height = 5)

# NMDS
t1_unwei$cal_ordination(method = "NMDS")
nmds_unwei <- t1_unwei$plot_ordination(
  plot_color             = TREATMENT_VAR,
  color_values           = my_color_palette,
  plot_type              = c("point", "ellipse"),
  centroid_segment_alpha = 1,
  point_alpha            = 1
) + theme_classic() + geom_vline(xintercept = 0, linetype = 2) + geom_hline(yintercept = 0, linetype = 2)
ggsave("nmds_unwei_unifrac.pdf", plot = nmds_unwei, width = 6, height = 5)</code></pre>
      </div>

      <h2>9️⃣ Group distance &amp; PERMANOVA</h2>
      <div class="codewrap">
        <button class="copybtn" onclick="copyCode('group-dist-perm')">📋 Copy</button>
        <pre><code id="group-dist-perm"># Weighted UniFrac
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

# Unweighted UniFrac
t1_unwei$cal_group_distance(within_group = TRUE)
t1_unwei$cal_group_distance_diff(method = "KW_dunn") #choose righst statistical test
write.csv(t1_unwei$res_group_distance_diff, "unwei_unifrac_distance_diff.csv")
dist_plot_unwei <- t1_unwei$plot_group_distance(
  boxplot_add = "mean",
  color_values = my_color_palette
) + theme(axis.text.x = element_text(angle = 90, vjust = 0.5, hjust = 1))
ggsave("unwei_unifrac_distance_boxplot.pdf", plot = dist_plot_unwei)
t1_unwei$cal_manova(manova_all = FALSE)
write.csv(t1_unwei$res_manova, "permanova_unwei_unifrac.csv")</code></pre>
      </div>

      <h2>🔟 Differential Abundance — LEfSe (all levels)</h2>
      <div class="codewrap">
        <button class="copybtn" onclick="copyCode('lefse-all')">📋 Copy</button>
        <pre><code id="lefse-all">tryCatch({
  t1_lefse_all <- trans_diff$new(
    dataset = dataset,
    method  = "lefse",
    group   = TREATMENT_VAR,
    taxa_level = "all",
    filter_thres = 0,
    alpha = 0.05,
    p_adjust_method = "none",
    lefse_subgroup = NULL
  )
  lefse_all <- t1_lefse_all$plot_diff_bar(keep_prefix = FALSE)
  pdf("lefse_all.pdf"); print(lefse_all); dev.off()
}, error = function(e) {
  message("Error in LEfSe (all levels): ", e$message)
})</code></pre>
      </div>

      <h2>🔟 Differential Abundance — LEfSe (by rank)</h2>
      <div class="codewrap">
        <button class="copybtn" onclick="copyCode('lefse-ranks')">📋 Copy</button>
        <pre><code id="lefse-ranks">for (rk in c("Phylum","Class","Order","Family","Genus")) {
  tryCatch({
    t1_lefse <- trans_diff$new(
      dataset = dataset,
      method  = "lefse",
      group   = TREATMENT_VAR,
      taxa_level = rk,
      filter_thres = 0,
      alpha = 0.05,
      p_adjust_method = "none"
    )
    p <- t1_lefse$plot_diff_bar(keep_prefix = FALSE)
    pdf(paste0("lefse_", rk, ".pdf")); print(p); dev.off()
  }, error = function(e) message("Error in LEfSe (", rk, "): ", e$message))
}</code></pre>
      </div>

      <h2>🔟 Differential Abundance — Metastat (all ranks)</h2>
      <div class="codewrap">
        <button class="copybtn" onclick="copyCode('metastat-all')">📋 Copy</button>
        <pre><code id="metastat-all">for (rk in c("Phylum","Class","Order","Family","Genus")) {
  t1_ms <- trans_diff$new(
    dataset    = dataset,
    method     = "metastat",
    group      = TREATMENT_VAR,
    taxa_level = rk
  )
  write.csv(t1_ms$res_diff,  paste0("metastat_diff_result_Treatment_", rk, ".csv"), row.names = FALSE)
  write.csv(t1_ms$res_abund, paste0("metastat_group_abund_Treatment_", rk, ".csv"), row.names = FALSE)
}</code></pre>
      </div>

      <h2>✅ Final Save</h2>
      <div class="codewrap">
        <button class="copybtn" onclick="copyCode('final-save')">📋 Copy</button>
        <pre><code id="final-save">writeLines(capture.output(sessionInfo()), SESSION_INFO)
save.image(file = FINAL_WORKSPACE)
message("All analyses complete. Session info and workspace saved.")</code></pre>
      </div>
    `;
  }

  // prebuild the page HTML so it's available to unlock()
  const PAGE_HTML = buildPageHTML();
  
  // Simple gate (no popups)
  const PASSWORD = "cqubioinfo2025";

 function unlock(e) {
    if (e && e.preventDefault) e.preventDefault();
    const input = document.getElementById('pw-input');
    const err = document.getElementById('pw-error');
    const card = document.getElementById('pw-card');
    const content = document.getElementById('content');
    if (!input || !content) return;

    if (input.value === PASSWORD) {
      content.innerHTML = PAGE_HTML;              // now defined ✅
      card.style.display = 'none';
      content.style.display = 'block';
      err.style.display = 'none';
    } else {
      err.style.display = 'block';
      input.focus();
      input.select();
    }
  }

  // Hook up events
  const formEl = document.getElementById('pw-form');
  const btnEl  = document.getElementById('pw-submit');
  formEl && formEl.addEventListener('submit', unlock);
  btnEl  && btnEl.addEventListener('click', unlock);
</script>
{% endraw %}
