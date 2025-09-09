---
layout: default
title: QIIME2 (Single-end) with DADA2
---

<style>
  .codewrap { position: relative; margin: 1rem 0; }
  .codewrap pre {
    background:#f6f8fa; padding:1em; border-radius:8px;
    overflow-x:auto; white-space:pre; /* preserve line breaks exactly */
  }
  .copybtn {
    position:absolute; top:10px; right:10px;
    background:#0366d6; color:#fff; border:0; padding:6px 10px;
    border-radius:6px; font-size:.85em; cursor:pointer;
  }
  h1,h2,h3 { margin-top: 1.2em; }
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
  // Copy helper (reused for all blocks)
  function copyCode(id) {
    const el = document.getElementById(id);
    if (!el) return;
    const text = el.innerText;
    navigator.clipboard.writeText(text).then(
      () => alert("✅ Code copied to clipboard!"),
      () => alert("⚠️ Copy failed. Please copy manually.")
    );
  }

  // Build the protected HTML once (so Jekyll/Liquid doesn't mangle it)
  const PAGE_HTML = `
    <h1>Analysis of 16S rRNA (Single-End) with QIIME 2 using DADA2</h1>
    <p>
      Ensure <code>manifest.txt</code> and <code>sample-metadata.txt</code> are in your working directory.
      This tutorial uses a shared SILVA classifier path.
    </p>
    <div class="note"><strong>Tip:</strong> Run each step in order. Adjust trimming parameters after
    inspecting the demux quality plots. Use <code>qiime tools view</code> or upload <code>.qzv</code> files to
    <a href="https://view.qiime2.org" target="_blank" rel="noopener">view.qiime2.org</a>.</div>

    <!-- 0) Env -->
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
# PRIMER_SEQ="GACTACNVGGGTATCTAATCC"    # example, if primer trimming is required later
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
      <pre><code id="step-1" class="language-bash">qiime tools import \\
  --type 'SampleData[SequencesWithQuality]' \\
  --input-path "$MANIFEST" \\
  --output-path demux.qza \\
  --input-format SingleEndFastqManifestPhred33V2

qiime demux summarize \\
  --i-data demux.qza \\
  --o-visualization demux.qzv

# Visualise locally (if supported) OR upload to https://view.qiime2.org
qiime tools view demux.qzv</code></pre>
    </div>

    <!-- 2) Cutadapt -->
    <h2>2) Remove adapter (and optionally primer) sequences</h2>
    <div class="codewrap">
      <button class="copybtn" onclick="copyCode('step-2')">📋 Copy</button>
      <pre><code id="step-2" class="language-bash">qiime cutadapt trim-single \\
  --i-demultiplexed-sequences demux.qza \\
  --p-cores $THREADS \\
  --p-adapter "$ADAPTER_SEQ" \\
  --o-trimmed-sequences trimmed_demux.qza

qiime demux summarize \\
  --i-data trimmed_demux.qza \\
  --o-visualization trimmed_demux.qzv</code></pre>
    </div>

    <!-- 3) DADA2 -->
    <h2>3) Denoise with DADA2 (single-end)</h2>
    <p class="muted">Tune <code>TRIM_LEFT</code>, <code>TRUNC_LEN</code>, and <code>MAX_EE</code> based on quality.</p>
    <div class="codewrap">
      <button class="copybtn" onclick="copyCode('step-3')">📋 Copy</button>
      <pre><code id="step-3" class="language-bash">qiime dada2 denoise-single \\
  --i-demultiplexed-seqs trimmed_demux.qza \\
  --p-trim-left $TRIM_LEFT \\
  --p-trunc-len  $TRUNC_LEN \\
  --p-max-ee     $MAX_EE \\
  --p-n-threads  $THREADS \\
  --o-table table.qza \\
  --o-representative-sequences rep-seqs.qza \\
  --o-denoising-stats denoising-stats.qza

qiime metadata tabulate \\
  --m-input-file denoising-stats.qza \\
  --o-visualization denoising-stats.qzv</code></pre>
    </div>

    <!-- 4) Summaries -->
    <h2>4) Feature table &amp; representative sequences summaries</h2>
    <div class="codewrap">
      <button class="copybtn" onclick="copyCode('step-4')">📋 Copy</button>
      <pre><code id="step-4" class="language-bash">qiime feature-table summarize \\
  --i-table table.qza \\
  --m-sample-metadata-file "$METADATA" \\
  --o-visualization table.qzv

qiime feature-table tabulate-seqs \\
  --i-data rep-seqs.qza \\
  --o-visualization rep-seqs.qzv</code></pre>
    </div>

    <!-- 5) Filter singles -->
    <h2>5) Filter features (remove singletons) &amp; resummarize</h2>
    <div class="codewrap">
      <button class="copybtn" onclick="copyCode('step-5')">📋 Copy</button>
      <pre><code id="step-5" class="language-bash"># Filter features with min frequency = 2
qiime feature-table filter-features \\
  --i-table table.qza \\
  --p-min-frequency 2 \\
  --o-filtered-table table-2.qza

qiime feature-table summarize \\
  --i-table table-2.qza \\
  --m-sample-metadata-file "$METADATA" \\
  --o-visualization table-2.qzv

# Filter representative sequences to match filtered table
qiime feature-table filter-seqs \\
  --i-data rep-seqs.qza \\
  --i-table table-2.qza \\
  --o-filtered-data rep-seqs-2.qza

qiime feature-table tabulate-seqs \\
  --i-data rep-seqs-2.qza \\
  --o-visualization rep-seqs-2.qzv</code></pre>
    </div>

    <!-- 6) Tree -->
    <h2>6) Generate phylogenetic tree</h2>
    <div class="codewrap">
      <button class="copybtn" onclick="copyCode('step-6')">📋 Copy</button>
      <pre><code id="step-6" class="language-bash">qiime phylogeny align-to-tree-mafft-fasttree \\
  --i-sequences rep-seqs-2.qza \\
  --output-dir phylogeny-align-to-tree-mafft-fasttree</code></pre>
    </div>

    <!-- 7) Taxonomy -->
    <h2>7) Taxonomic classification</h2>
    <div class="codewrap">
      <button class="copybtn" onclick="copyCode('step-7')">📋 Copy</button>
      <pre><code id="step-7" class="language-bash">qiime feature-classifier classify-sklearn \\
  --i-classifier "$CLASSIFIER" \\
  --i-reads rep-seqs-2.qza \\
  --o-classification taxonomy.qza

qiime metadata tabulate \\
  --m-input-file taxonomy.qza \\
  --o-visualization taxonomy.qzv</code></pre>
    </div>

    <!-- 8) Barplots -->
    <h2>8) Taxa bar plots</h2>
    <div class="codewrap">
      <button class="copybtn" onclick="copyCode('step-8')">📋 Copy</button>
      <pre><code id="step-8" class="language-bash">qiime taxa barplot \\
  --i-table table-2.qza \\
  --i-taxonomy taxonomy.qza \\
  --m-metadata-file "$METADATA" \\
  --o-visualization taxa-bar-plots.qzv</code></pre>
    </div>

    <!-- 9) Alpha rarefaction -->
    <h2>9) Alpha rarefaction</h2>
    <div class="codewrap">
      <button class="copybtn" onclick="copyCode('step-9')">📋 Copy</button>
      <pre><code id="step-9" class="language-bash">qiime diversity alpha-rarefaction \\
  --i-table table-2.qza \\
  --i-phylogeny phylogeny-align-to-tree-mafft-fasttree/rooted_tree.qza \\
  --p-max-depth $MAX_DEPTH \\
  --m-metadata-file "$METADATA" \\
  --o-visualization alpha-rarefaction.qzv</code></pre>
    </div>
  `;

  // Simple gate (no popups)
  const PASSWORD = "cqubioinfo2025";

  function unlock() {
    const input = document.getElementById('pw-input');
    const err = document.getElementById('pw-error');
    const card = document.getElementById('pw-card');
    const content = document.getElementById('content');
    if (!input || !content) return;

    if (input.value === PASSWORD) {
      content.innerHTML = PAGE_HTML;
      card.style.display = 'none';
      content.style.display = 'block';
    } else {
      err.style.display = 'block';
      input.focus();
      input.select();
    }
  }

  // Hook up events
  document.getElementById('pw-form')?.addEventListener('submit', unlock);
  document.getElementById('pw-submit')?.addEventListener('click', unlock);
</script>
{% endraw %}
