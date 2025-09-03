---
layout: default
title: QIIME 2 (Single-end) with DADA2 — Step-by-step
---

<style>
  .codewrap { position: relative; margin: 1rem 0; }
  .codewrap pre { background:#f6f8fa; padding:1em; border-radius:8px; overflow:auto; }
  .copybtn {
    position:absolute; top:10px; right:10px;
    background:#0366d6; color:#fff; border:0; padding:6px 10px;
    border-radius:6px; font-size:.85em; cursor:pointer;
  }
  h2, h3 { margin-top: 1.2em; }
  .muted { color:#666; }
  .note { background:#fff7cc; border:1px solid #ffe58a; padding:.6em .8em; border-radius:6px; }
</style>

<div id="gate"></div>
<div id="content" style="display:none;"></div>

<script>
  // Reusable copy helper (used by all blocks)
  function copyCode(id) {
    const el = document.getElementById(id);
    if (!el) return;
    const text = el.innerText;
    navigator.clipboard.writeText(text).then(
      () => alert("✅ Code copied to clipboard!"),
      () => alert("⚠️ Copy failed. Please copy manually.")
    );
  }

  (function () {
    const correctPassword = "cqubioinfo2025";
    const userInput = prompt("🔒 Enter password to access this training page:");
    const gate = document.getElementById("gate");
    const content = document.getElementById("content");

    if (userInput === correctPassword) {
      content.innerHTML = `
        <h1>Analysis of 16S rRNA (Single-End) with QIIME 2 using DADA2</h1>
        <p>
          Ensure <code>manifest.txt</code> and <code>sample-metadata.txt</code> are in your working directory. 
          This tutorial uses a shared SILVA classifier path.
        </p>

        <div class="note"><strong>Tip:</strong> Run each step in order. Adjust trimming parameters after inspecting the demux quality plots.</div>

        <!-- 0) Load QIIME2 / Activate env -->
        <h2>0) Load QIIME 2 and activate environment</h2>
        <div class="codewrap">
          <button class="copybtn" onclick="copyCode('step-env')">📋 Copy</button>
          <pre><code id="step-env" class="language-bash"># Change QIIME_ENV when QIIME2 is updated
QIIME_ENV="/project/2025-sharma-cqu-bioinfo/envs/qiime2-amplicon-2024.10"
module load Anaconda3/2024.06-1
conda activate "$QIIME_ENV"</code></pre>
        </div>

        <!-- Variables -->
        <h2>Variables (edit per project)</h2>
        <div class="codewrap">
          <button class="copybtn" onclick="copyCode('step-vars')">📋 Copy</button>
          <pre><code id="step-vars" class="language-bash">USERNAME="type your user name here"
MANIFEST="/project/2025-sharma-cqu-bioinfo/training/manifest.txt"
METADATA="/project/2025-sharma-cqu-bioinfo/training/sample-metadata.txt"
ADAPTER_SEQ="AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT"
# PRIMER_SEQ="GACTACNVGGGTATCTAATCC"    # example, if you need primer trimming later
CLASSIFIER="/project/2025-sharma-cqu-bioinfo/training/silva-138-99-nb-classifier.qza"
THREADS=24
MAX_DEPTH=3000

# Define after checking quality plots (demux.qzv) before DADA2:
TRIM_LEFT=21
TRUNC_LEN=250
MAX_EE=2.0

# Workspace
mkdir -p /home/$USERNAME/data/q2_training
WORKDIR="/home/$USERNAME/data/q2_training"
cd "$WORKDIR"</code></pre>
        </div>

        <!-- 1) Import -->
        <h2>1) Import data &amp; summarize</h2>
        <div class="codewrap">
          <button class="copybtn" onclick="copyCode('step-1')">📋 Copy</button>
          <pre><code id="step-1" class="language-bash">qiime tools import \
  --type 'SampleData[SequencesWithQuality]' \
  --input-path "$MANIFEST" \
  --output-path demux.qza \
  --input-format SingleEndFastqManifestPhred33V2

qiime demux summarize \
  --i-data demux.qza \
  --o-visualization demux.qzv

# Visualise locally (if supported) OR upload to https://view.qiime2.org
qiime tools view demux.qzv</code></pre>
        </div>

        <!-- 2) Cutadapt -->
        <h2>2) Remove adapter (and optionally primer) sequences</h2>
        <div class="codewrap">
          <button class="copybtn" onclick="copyCode('step-2')">📋 Copy</button>
          <pre><code id="step-2" class="language-bash">qiime cutadapt trim-single \
  --i-demultiplexed-sequences demux.qza \
  --p-cores $THREADS \
  --p-adapter "$ADAPTER_SEQ" \
  --o-trimmed-sequences trimmed_demux.qza

qiime demux summarize \
  --i-data trimmed_demux.qza \
  --o-visualization trimmed_demux.qzv</code></pre>
        </div>

        <!-- 3) DADA2 -->
        <h2>3) Denoise with DADA2 (single-end)</h2>
        <p class="muted">Tune <code>TRIM_LEFT</code>, <code>TRUNC_LEN</code>, and <code>MAX_EE</code> based on quality.</p>
        <div class="codewrap">
          <button class="copybtn" onclick="copyCode('step-3')">📋 Copy</button>
          <pre><code id="step-3" class="language-bash">qiime dada2 denoise-single \
  --i-demultiplexed-seqs trimmed_demux.qza \
  --p-trim-left $TRIM_LEFT \
  --p-trunc-len  $TRUNC_LEN \
  --p-max-ee     $MAX_EE \
  --p-n-threads  $THREADS \
  --o-table table.qza \
  --o-representative-sequences rep-seqs.qza \
  --o-denoising-stats denoising-stats.qza

qiime metadata tabulate \
  --m-input-file denoising-stats.qza \
  --o-visualization denoising-stats.qzv</code></pre>
        </div>

        <!-- 4) Summaries -->
        <h2>4) Feature table &amp; representative sequences summaries</h2>
        <div class="codewrap">
          <button class="copybtn" onclick="copyCode('step-4')">📋 Copy</button>
          <pre><code id="step-4" class="language-bash">qiime feature-table summarize \
  --i-table table.qza \
  --m-sample-metadata-file "$METADATA" \
  --o-visualization table.qzv

qiime feature-table tabulate-seqs \
  --i-data rep-seqs.qza \
  --o-visualization rep-seqs.qzv</code></pre>
        </div>

        <!-- 5) Filter singles -->
        <h2>5) Filter features (remove singletons) &amp; resummarize</h2>
        <div class="codewrap">
          <button class="copybtn" onclick="copyCode('step-5')">📋 Copy</button>
          <pre><code id="step-5" class="language-bash"># Filter features with min frequency = 2
qiime feature-table filter-features \
  --i-table table.qza \
  --p-min-frequency 2 \
  --o-filtered-table table-2.qza

qiime feature-table summarize \
  --i-table table-2.qza \
  --m-sample-metadata-file "$METADATA" \
  --o-visualization table-2.qzv

# Filter representative sequences to match filtered table
qiime feature-table filter-seqs \
  --i-data rep-seqs.qza \
  --i-table table-2.qza \
  --o-filtered-data rep-seqs-2.qza

qiime feature-table tabulate-seqs \
  --i-data rep-seqs-2.qza \
  --o-visualization rep-seqs-2.qzv</code></pre>
        </div>

        <!-- 6) Tree -->
        <h2>6) Generate phylogenetic tree</h2>
        <div class="codewrap">
          <button class="copybtn" onclick="copyCode('step-6')">📋 Copy</button>
          <pre><code id="step-6" class="language-bash">qiime phylogeny align-to-tree-mafft-fasttree \
  --i-sequences rep-seqs-2.qza \
  --output-dir phylogeny-align-to-tree-mafft-fasttree</code></pre>
        </div>

        <!-- 7) Taxonomy -->
        <h2>7) Taxonomic classification</h2>
        <div class="codewrap">
          <button class="copybtn" onclick="copyCode('step-7')">📋 Copy</button>
          <pre><code id="step-7" class="language-bash">qiime feature-classifier classify-sklearn \
  --i-classifier "$CLASSIFIER" \
  --i-reads rep-seqs-2.qza \
  --o-classification taxonomy.qza

qiime metadata tabulate \
  --m-input-file taxonomy.qza \
  --o-visualization taxonomy.qzv</code></pre>
        </div>

        <!-- 8) Barplots -->
        <h2>8) Taxa bar plots</h2>
        <div class="codewrap">
          <button class="copybtn" onclick="copyCode('step-8')">📋 Copy</button>
          <pre><code id="step-8" class="language-bash">qiime taxa barplot \
  --i-table table-2.qza \
  --i-taxonomy taxonomy.qza \
  --m-metadata-file "$METADATA" \
  --o-visualization taxa-bar-plots.qzv</code></pre>
        </div>

        <!-- 9) Alpha rarefaction -->
        <h2>9) Alpha rarefaction</h2>
        <div class="codewrap">
          <button class="copybtn" onclick="copyCode('step-9')">📋 Copy</button>
          <pre><code id="step-9" class="language-bash">qiime diversity alpha-rarefaction \
  --i-table table-2.qza \
  --i-phylogeny phylogeny-align-to-tree-mafft-fasttree/rooted_tree.qza \
  --p-max-depth $MAX_DEPTH \
  --m-metadata-file "$METADATA" \
  --o-visualization alpha-rarefaction.qzv</code></pre>
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
