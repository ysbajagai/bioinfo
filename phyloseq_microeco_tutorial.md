---
layout: default
title: 16S Phyloseq & Microeco Analysis Tutorial
---

<style>
  .codewrap { position: relative; margin: 1rem 0; }
  .codewrap pre {
    background: #f6f8fa; padding: 1em; border-radius: 8px; overflow: auto;
  }
  .copybtn {
    position: absolute; top: 10px; right: 10px;
    background: #0366d6; color: #fff; border: 0; padding: 6px 10px;
    border-radius: 6px; font-size: 0.85em; cursor: pointer;
  }
  h2, h3 { margin-top: 1.2em; }
  .muted { color:#666; }
</style>

<div id="gate"></div>
<div id="content" style="display:none;"></div>

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

  // 2) Password gate
  (function () {
    const correctPassword = "cqubioinfo2025";
    const userInput = prompt("🔒 Enter password to access this training page:");
    const gate = document.getElementById("gate");
    const content = document.getElementById("content");

    if (userInput === correctPassword) {
      // 3) Build HTML content WITHOUT any <script> tags or markdown fences
      content.innerHTML = `
        <h1>16S rRNA Sequencing Data Analysis in R using Phyloseq and Microeco</h1>
        <p>
          This tutorial provides a complete R-based workflow for analyzing 16S rRNA sequencing data using
          <code>qiime2R</code>, <code>phyloseq</code>, and <code>microeco</code>.
        </p>

        <hr />
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
METADATA_SHEET <- "Sheet1"  # The sheet name where metadata is stored
### Names of Qiime2 artifacts to import
FEATURE_TABLE_QZA <- "table.qza"
TAXONOMY_QZA      <- "taxonomy.qza"
TREE_QZA          <- "rooted_tree.qza"
### Optional .RData output file name
PSEQ_RDATA <- "XXX_pseq.RData"
### Directories and filenames for microeco step
MECO_DIR <- "./meco"
### Rarefaction depth for microeco
RAREFACTION_DEPTH <-   # you need to come back here later
### For alpha/beta diversity, bar plots, etc.
TREATMENT_VAR <- "Treatment"   # The name of the variable in your sample_data
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

        <h2>7️⃣ Bar plots for relative abundance</h2>
        <p class="muted">Examples for Phylum/Class/Order/Family/Genus with per-sample and group-mean plots.</p>
        <div class="codewrap">
          <button class="copybtn" onclick="copyCode('barplot-taxonomic-ranks')">📋 Copy</button>
          <pre><code id="barplot-taxonomic-ranks"># Example: PHYLUM per-sample faceted by Treatment
t1_Phylum <- trans_abund$new(dataset = dataset, taxrank = "Phylum", ntaxa = 10)
bar_plot_Phylum_Treatment <- t1_Phylum$plot_bar(
  color_values = paletteer::paletteer_d("ggthemes::Classic_20"),
  bar_full     = TRUE,
  others_color = "grey90",
  facet        = TREATMENT_VAR,
  strip_text   = 18,
  legend_text_italic = FALSE,
  xtext_angle  = 60,
  xtext_size   = 16
)
ggsave("bar_plot_Phylum_Treatment.pdf", plot = bar_plot_Phylum_Treatment, device = "pdf")
ggsave("bar_plot_Phylum_Treatment.png", plot = bar_plot_Phylum_Treatment, device = "png")

# Repeat analogous blocks for Class/Order/Family/Genus ...</code></pre>
        </div>

        <h2>8️⃣ Alpha &amp; Beta Diversity</h2>
        <div class="codewrap">
          <button class="copybtn" onclick="copyCode('alpha-beta')">📋 Copy</button>
          <pre><code id="alpha-beta"># Alpha diversity
t1_alpha <- trans_alpha$new(dataset = dataset, group = TREATMENT_VAR)
alpha_summary <- t1_alpha$data_stat
write.csv(alpha_summary, "alpha_summary.csv")
t1_alpha$cal_diff(method = "KW")
write.csv(t1_alpha$res_diff, "alpha_diff_KW.csv")

# Example plotting loop omitted for brevity...

# Beta diversity (Weighted/Unweighted UniFrac)
t1_wei  <- trans_beta$new(dataset = dataset, group = TREATMENT_VAR, measure = "wei_unifrac")
t1_unwei<- trans_beta$new(dataset = dataset, group = TREATMENT_VAR, measure = "unwei_unifrac")
t1_wei$cal_ordination(method = "PCoA")
pcoa_wei <- t1_wei$plot_ordination(plot_color = TREATMENT_VAR)
ggsave("pcoa_wei_unifrac.pdf", plot = pcoa_wei, width = 6, height = 5)</code></pre>
        </div>

        <h2>9️⃣ Group distance &amp; PERMANOVA</h2>
        <div class="codewrap">
          <button class="copybtn" onclick="copyCode('group-dist-perm')">📋 Copy</button>
          <pre><code id="group-dist-perm"># Weighted UniFrac
t1_wei$cal_group_distance(within_group = TRUE)
t1_wei$cal_group_distance_diff(method = "KW")
write.csv(t1_wei$res_group_distance_diff, "wei_unifrac_distance_diff.csv")
t1_wei$cal_manova(manova_all = FALSE)
write.csv(t1_wei$res_manova, "permanova_wei_unifrac.csv")

# Unweighted UniFrac
t1_unwei$cal_group_distance(within_group = TRUE)
t1_unwei$cal_group_distance_diff(method = "KW")
write.csv(t1_unwei$res_group_distance_diff, "unwei_unifrac_distance_diff.csv")
t1_unwei$cal_manova(manova_all = FALSE)
write.csv(t1_unwei$res_manova, "permanova_unwei_unifrac.csv")</code></pre>
        </div>

        <h2>🔟 Differential Abundance (LEfSe &amp; Metastat)</h2>
        <div class="codewrap">
          <button class="copybtn" onclick="copyCode('diff-taxa')">📋 Copy</button>
          <pre><code id="diff-taxa"># LEfSe (all levels example)
tryCatch({
  t1_lefse_all <- trans_diff$new(
    dataset = dataset, method = "lefse", group = TREATMENT_VAR,
    taxa_level = "all", filter_thres = 0, alpha = 0.05, p_adjust_method = "none"
  )
  lefse_all <- t1_lefse_all$plot_diff_bar(keep_prefix = FALSE)
  pdf("lefse_all.pdf"); print(lefse_all); dev.off()
}, error = function(e) message("Error in LEfSe (all levels): ", e$message))

# Metastat (Genus example)
t1_Treatment_metastat <- trans_diff$new(
  dataset = dataset, method = "metastat", group = TREATMENT_VAR, taxa_level = "Genus"
)
write.csv(t1_Treatment_metastat$res_diff,  "metastat_diff_result_Treatment.csv")
write.csv(t1_Treatment_metastat$res_abund, "metastat_group_abund_Treatment.csv")</code></pre>
        </div>

        <h2>✅ Final Save</h2>
        <div class="codewrap">
          <button class="copybtn" onclick="copyCode('final-save')">📋 Copy</button>
          <pre><code id="final-save">writeLines(capture.output(sessionInfo()), SESSION_INFO)
save.image(file = FINAL_WORKSPACE)
message("All analyses complete. Session info and workspace saved.")</code></pre>
        </div>
      `;
      gate.style.display = "none";
      content.style.display = "block";
    } else {
      gate.innerHTML = `
        <div style="text-align:center; padding-top:50px; font-family:sans-serif;">
          <h2 style="color:#c00;">🚫 Access Denied</h2>
          <p style="font-size:18px;">This content is restricted.</p>
          <p style="font-size:16px;">
            If you would like access, please contact:<br>
            <a href="mailto:y.sharmabajagai@cqu.edu.au">y.sharmabajagai@cqu.edu.au</a>
          </p>
        </div>
      `;
    }
  })();
</script>
