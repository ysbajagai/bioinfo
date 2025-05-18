<div id="protected-content" style="display:none;">

---
layout: default
title: Tools and Pipelines
---

# 🧪 Bioinformatics Tools & Pipelines

An introduction to commonly used bioinformatics tools and how to automate workflows.

## Tools Covered

- FASTQC
- Trimmomatic
- BWA & Bowtie2
- SAMtools
- Snakemake & Nextflow

</div> <!-- End of protected content div -->

<script>
  var correctPassword = "cqubioinfo2025";
  var userInput = prompt("🔒 Enter password to access this training page:");
  if (userInput === correctPassword) {
    document.getElementById("protected-content").style.display = "block";
  } else {
    document.body.innerHTML = `
      <div style="text-align:center; padding-top:50px; font-family:sans-serif;">
        <h2 style="color:#c00;">🚫 Access Denied</h2>
        <p style="font-size:18px;">This content is restricted.</p>
        <p style="font-size:16px;">
          If you would like access, please contact:<br>
          <a href="mailto:y.sharmabajagai@cqu.edu.au">y.sharmabajagai@cqu.edu.au</a>
        </p>
      </div>`;
  }
</script>
