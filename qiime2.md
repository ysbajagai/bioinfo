<script>
  const correctPassword = "cqubioinfo2025";
  const userInput = prompt("🔒 Enter password to access this training page:");

  if (userInput === correctPassword) {
    document.write(`
<pre><code class="language-bash">
# Analysis of 16S rRNA sequencing data (single-end) with QIIME2 using DADA2 plugin
#Ensure you have "manifest.txt" file, and "sample-metadata.txt" files  in your working directory. You don't need classifier if you use the following code.

# ======================
# Load QIIME2 module - change the QIIME_ENV when qiime2 is updated
# ======================

QIIME_ENV="/project/2025-sharma-cqu-bioinfo/envs/qiime2-amplicon-2024.10"
module load Anaconda3/2024.06-1
conda activate "$QIIME_ENV"

# ======================
# Define Variables (change for each project)
# ======================

USERNAME="type your user name here"
MANIFEST="/project/2025-sharma-cqu-bioinfo/training/manifest.txt"
METADATA="/project/2025-sharma-cqu-bioinfo/training/sample-metadata.txt"
ADAPTER_SEQ="AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT"
#PRIMER_SEQ="GACTACNVGGGTATCTAATCC"
CLASSIFIER="/project/2025-sharma-cqu-bioinfo/training/silva-138-99-nb-classifier.qza"
THREADS=24
MAX_DEPTH=3000

#define the following after checking the quality plot before running dada2

TRIM_LEFT=21
TRUNC_LEN=250
MAX_EE=2.0

mkdir /home/$USERNAME/data/q2_training
WORKDIR="/home/$USERNAME/data/q2_training"

cd $WORKDIR

# ======================
# Step 1: Import Data
# ======================

qiime tools import \\
  --type 'SampleData[SequencesWithQuality]' \\
  --input-path "$MANIFEST" \\
  --output-path demux.qza \\
  --input-format SingleEndFastqManifestPhred33V2

qiime demux summarize \\
  --i-data demux.qza \\
  --o-visualization demux.qzv

  # demux.qzv file (or any other .qzv files) can be visualised as below

qiime tools view demux.qzv
  
#if your command line didn't support direct visualisation from the command line, save the file locally and visualise with www.view.qiime2.org in your browser

# ======================
# Step 2: Remove Adapter and primer Sequences
# ======================

qiime cutadapt trim-single \\
  --i-demultiplexed-sequences demux.qza \\
  --p-cores $THREADS \\
  --p-adapter "$ADAPTER_SEQ" \\
  --o-trimmed-sequences trimmed_demux.qza

qiime demux summarize \\
  --i-data trimmed_demux.qza \\
  --o-visualization trimmed_demux.qzv

# ======================
# Step 3: Denoise with DADA2
# ======================
qiime dada2 denoise-single \\
  --i-demultiplexed-seqs trimmed_demux.qza \\
  --p-trim-left $TRIM_LEFT \\
  --p-trunc-len $TRUNC_LEN \\
  --p-max-ee $MAX_EE \\
  --p-n-threads $THREADS \\
  --o-table table.qza \\
  --o-representative-sequences rep-seqs.qza \\
  --o-denoising-stats denoising-stats.qza

qiime metadata tabulate \\
  --m-input-file denoising-stats.qza \\
  --o-visualization denoising-stats.qzv

# ======================
# Step 4: Feature Table and Representative Sequences Summaries
# ======================
qiime feature-table summarize \\
  --i-table table.qza \\
  --m-sample-metadata-file "$METADATA" \\
  --o-visualization table.qzv

qiime feature-table tabulate-seqs \\
  --i-data rep-seqs.qza \\
  --o-visualization rep-seqs.qzv

# ======================
# Step 5: Filter feature-table and summarise - remove singletons
# ======================

qiime feature-table filter-features \\
  --i-table table.qza \\
  --p-min-frequency 2 \\
  --o-filtered-table table-2.qza

qiime feature-table summarize \\
  --i-table table-2.qza \\
  --m-sample-metadata-file "$METADATA" \\
  --o-visualization table-2.qzv
  
##corresponding representative sequence files after removing singletons

qiime feature-table filter-seqs \\
  --i-data rep-seqs.qza \\
  --i-table table-2.qza \\
  --o-filtered-data rep-seqs-1.qza

qiime feature-table tabulate-seqs \\
  --i-data rep-seqs-2.qza \\
  --o-visualization rep-seqs-2.qzv

# ======================
# Step 6: Generate Phylogenetic Tree
# ======================
qiime phylogeny align-to-tree-mafft-fasttree \\
  --i-sequences rep-seqs-2.qza \\
  --output-dir phylogeny-align-to-tree-mafft-fasttree

# ======================
# Step 7: Taxonomic Classification
# ======================
qiime feature-classifier classify-sklearn \\
  --i-classifier "$CLASSIFIER" \\
  --i-reads rep-seqs-2.qza \\
  --o-classification taxonomy.qza

qiime metadata tabulate \\
  --m-input-file taxonomy.qza \\
  --o-visualization taxonomy.qzv

# ======================
# Step 8: Taxa Bar Plots
# ======================
qiime taxa barplot \\
  --i-table table-2.qza \\
  --i-taxonomy taxonomy.qza \\
  --m-metadata-file "$METADATA" \\
  --o-visualization taxa-bar-plots.qzv

# ======================
# Step 9: Alpha Rarefaction
# ======================
qiime diversity alpha-rarefaction \\
  --i-table table-2.qza \\
  --i-phylogeny phylogeny-align-to-tree-mafft-fasttree/rooted_tree.qza \\
  --p-max-depth $MAX_DEPTH \\
  --m-metadata-file "$METADATA" \\
  --o-visualization alpha-rarefaction.qzv
</code></pre>
`);
  } else {
    document.write(`
      <div style="text-align:center; padding-top:50px; font-family:sans-serif;">
        <h2 style="color:#c00;">🚫 Access Denied</h2>
        <p style="font-size:18px;">This content is restricted.</p>
        <p style="font-size:16px;">
          If you would like access, please contact:<br>
          <a href="mailto:y.sharmabajagai@cqu.edu.au">y.sharmabajagai@cqu.edu.au</a>
        </p>
      </div>
    `);
  }
</script>
