<script>
  const correctPassword = "cqubioinfo2025";
  const userInput = prompt("🔒 Enter password to access this training page:");

  if (userInput === correctPassword) {
    document.write(`
<pre><code class="language-bash">
# Analysis of 16S rRNA sequencing data (single-end) with QIIME2 using DADA2 plugin

# Load QIIME2 module
module load Anaconda3/2024.06-1
conda activate /project/2025-sharma-cqu-bioinfo/envs/qiime2-amplicon-2024.10

# ======================
# Define Variables (change for each project)
# ======================

MANIFEST="manifest.txt"
METADATA="sample-metadata.txt"
ADAPTER_SEQ="AGATCGGAAGAGCGTCGTGTAGGGAAAGAGTGT"
#PRIMER_SEQ="GACTACNVGGGTATCTAATCC"
CLASSIFIER="/project/2025-sharma-cqu-bioinfo/training/silva-138-99-nb-classifier.qza"
THREADS=24
MAX_DEPTH=5000

#define the following after checking the quality plot before running dada2

TRIM_LEFT=21
TRUNC_LEN=250
MAX_EE=2.0

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
# Step 4: Filter feature-table
# ======================

qiime feature-table filter-features \
  --i-table table.qza \
  --p-min-frequency 5 \
  --p-min-samples 3
  --o-filtered-table filtered-table.qza

##keep only filtered-table ASVs in rep-seqs

qiime feature-table filter-seqs \
	--i-data rep-seqs.qza \
	--i-table filtered-table.qza \
	--o-filtered-data filtered-rep-seqs.qza

# ======================
# Step 5: Feature Table and Representative Sequences Summaries
# ======================
qiime feature-table summarize \
  --i-table filtered-table.qza \
  --m-sample-metadata-file "$METADATA" \
  --o-visualization filtered-table.qzv

qiime feature-table tabulate-seqs \
	--i-data filtered-rep-seqs.qza \
	--o-visualization filtered-rep-seqs.qzv

# ======================
# Step 6: Generate Phylogenetic Tree
# ======================
qiime phylogeny align-to-tree-mafft-fasttree \\
  --i-sequences filtered-rep-seqs.qza \\
  --output-dir phylogeny-align-to-tree-mafft-fasttree

# ======================
# Step 7: Taxonomic Classification
# ======================
qiime feature-classifier classify-sklearn \\
  --i-classifier "$CLASSIFIER" \\
  --i-reads filtered-rep-seqs.qza \\
  --o-classification taxonomy.qza

qiime metadata tabulate \\
  --m-input-file taxonomy.qza \\
  --o-visualization taxonomy.qzv

# ======================
# Step 8: Taxa Bar Plots
# ======================
qiime taxa barplot \\
  --i-table filtered-table.qza \\
  --i-taxonomy taxonomy.qza \\
  --m-metadata-file "$METADATA" \\
  --o-visualization taxa-bar-plots.qzv

# ======================
# Step 9: Alpha Rarefaction
# ======================
qiime diversity alpha-rarefaction \\
  --i-table filtered-table.qza \\
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
