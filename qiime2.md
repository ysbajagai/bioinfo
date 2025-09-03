---
layout: default
title: QIIME2 (Single-end) with DADA2
---

<style>
  .codewrap { position: relative; margin: 1rem 0; }
  .codewrap pre {
    background:#f6f8fa; padding:1em; border-radius:8px; 
    overflow-x:auto; white-space:pre; /* preserve newlines */
  }
  .copybtn {
    position:absolute; top:10px; right:10px;
    background:#0366d6; color:#fff; border:0; padding:6px 10px;
    border-radius:6px; font-size:.85em; cursor:pointer;
  }
</style>

<div id="gate"></div>
<div id="content" style="display:none;"></div>

<script>
function copyCode(id) {
  const el = document.getElementById(id);
  if (!el) return;
  navigator.clipboard.writeText(el.innerText).then(
    ()=>alert("✅ Code copied to clipboard!"),
    ()=>alert("⚠️ Copy failed.")
  );
}

(function () {
  const correctPassword = "cqubioinfo2025";
  const userInput = prompt("🔒 Enter password to access this training page:");
  const gate = document.getElementById("gate");
  const content = document.getElementById("content");

  if (userInput === correctPassword) {
    content.innerHTML = `
      <h1>QIIME2 (Single-end) DADA2 Workflow</h1>

      <h2>0) Load QIIME2 / Activate env</h2>
      <div class="codewrap">
        <button class="copybtn" onclick="copyCode('step-env')">📋 Copy</button>
        <pre><code id="step-env">QIIME_ENV="/project/2025-sharma-cqu-bioinfo/envs/qiime2-amplicon-2024.10"
module load Anaconda3/2024.06-1
conda activate "$QIIME_ENV"</code></pre>
      </div>

      <h2>Variables</h2>
      <div class="codewrap">
        <button class="copybtn" onclick="copyCode('step-vars')">📋 Copy</button>
        <pre><code id="step-vars">USERNAME="your username"
MANIFEST="/project/2025-sharma-cqu-bioinfo/training/manifest.txt"
METADATA="/project/2025-sharma-cqu-bioinfo/training/sample-metadata.txt"
ADAPTER_SEQ="AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT"
CLASSIFIER="/project/2025-sharma-cqu-bioinfo/training/silva-138-99-nb-classifier.qza"
THREADS=24
MAX_DEPTH=3000
TRIM_LEFT=21
TRUNC_LEN=250
MAX_EE=2.0

mkdir -p /home/$USERNAME/data/q2_training
WORKDIR="/home/$USERNAME/data/q2_training"
cd "$WORKDIR"</code></pre>
      </div>

      <h2>1) Import data & summarize</h2>
      <div class="codewrap">
        <button class="copybtn" onclick="copyCode('step-1')">📋 Copy</button>
        <pre><code id="step-1">qiime tools import \\
  --type 'SampleData[SequencesWithQuality]' \\
  --input-path "$MANIFEST" \\
  --output-path demux.qza \\
  --input-format SingleEndFastqManifestPhred33V2

qiime demux summarize \\
  --i-data demux.qza \\
  --o-visualization demux.qzv</code></pre>
      </div>

      <h2>2) Cutadapt trimming</h2>
      <div class="codewrap">
        <button class="copybtn" onclick="copyCode('step-2')">📋 Copy</button>
        <pre><code id="step-2">qiime cutadapt trim-single \\
  --i-demultiplexed-sequences demux.qza \\
  --p-cores $THREADS \\
  --p-adapter "$ADAPTER_SEQ" \\
  --o-trimmed-sequences trimmed_demux.qza</code></pre>
      </div>

      <!-- Continue similarly for steps 3–9 … -->
    `;
    gate.style.display = "none";
    content.style.display = "block";
  } else {
    gate.innerHTML = \`
      <div style="text-align:center; padding-top:50px; font-family:sans-serif;">
        <h2 style="color:#c00;">🚫 Access Denied</h2>
        <p>This content is restricted. Contact <a href="mailto:y.sharmabajagai@cqu.edu.au">y.sharmabajagai@cqu.edu.au</a></p>
      </div>
    \`;
  }
})();
</script>
