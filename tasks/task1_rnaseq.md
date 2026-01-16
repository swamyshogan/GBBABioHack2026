# Task 1: RNA-Seq Analysis Pipeline ⭐

**Difficulty:** Beginner-Friendly  
**Estimated Time:** 8-12 hours  
**Dataset Size:** ~800 MB

---

## 🎯 The Challenge

Build an RNA-seq analysis pipeline that takes raw sequencing reads and identifies genes that are differentially expressed between two conditions.

**In simple terms:** Given RNA-seq data from control and treatment samples, find which genes changed.

---

## 📦 What You'll Get

**Dataset:**
- 6 paired-end RNA-seq samples (3 control, 3 treatment)
- Human transcriptome data (chromosome 1 only for speed)
- Reference genome (FASTA) and gene annotations (GTF)
- Sample metadata sheet

**Download:** [LINK_TO_BE_PROVIDED]

---

## ✅ Minimum Requirements

Your pipeline must complete these core steps:

1. **Quality check** the raw sequencing data
2. **Align** reads to a reference genome
3. **Count** how many reads map to each gene
4. **Identify** differentially expressed genes
5. **Visualize** the results

That's it. How you do it is up to you.

---

## 💡 Ideas & Inspiration

### Possible Approaches

**Quality Control:**
- FastQC for basic quality metrics
- MultiQC to summarize reports
- Optional: Trim adapters with Trimmomatic or fastp

**Alignment:**
- STAR (standard for RNA-seq, memory-hungry)
- HISAT2 (lighter on memory)
- Salmon (ultra-fast, pseudo-alignment)
- Kallisto (also fast, different algorithm)

**Quantification:**
- HTSeq (classic, straightforward)
- featureCounts (fast, part of Subread)
- RSEM (if using STAR)
- Built-in to Salmon/Kallisto

**Differential Expression:**
- DESeq2 (most popular, in R)
- edgeR (also R, similar to DESeq2)
- limma-voom (another R option)

**Visualization:**
- Volcano plot (fold change vs p-value)
- MA plot (mean expression vs fold change)
- Heatmap (top genes across samples)
- PCA plot (sample clustering)

### Things to Consider

**Do you want to:**
- Optimize for speed? → Use Salmon or Kallisto
- Follow traditional best practices? → Use STAR + HTSeq + DESeq2
- Minimize memory usage? → Use HISAT2 instead of STAR
- Add extra QC steps? → Include post-alignment QC
- Compare different tools? → Run multiple aligners and compare

**Bonus Ideas:**
- Filter low-count genes before differential expression
- Adjust for batch effects
- Perform GO enrichment analysis on significant genes
- Create an interactive HTML report
- Add pathway analysis (KEGG, Reactome)
- Compare multiple differential expression tools

---

## 📊 Example Workflow

Here's **one possible approach** (but definitely not the only way):


Raw FastQ files
↓
FastQC (quality control)
↓
Trimmomatic (optional: trim adapters)
↓
STAR (alignment to genome)
↓
featureCounts (count reads per gene)
↓
DESeq2 (differential expression in R)
↓
Plots (volcano, heatmap, PCA)
↓
MultiQC (aggregate all QC reports)

But you could also:
- Skip trimming if quality is good
- Use Salmon instead of STAR → no alignment needed
- Use HTSeq instead of featureCounts
- Add additional filtering steps
- Create more sophisticated visualizations

---

## 🎨 Make It Your Own

### Stand Out With:

**Creative Analysis:**
- Cluster genes by expression pattern
- Identify co-expressed gene modules
- Predict transcription factor activity
- Find enriched motifs in promoters

**Better Visualizations:**
- Interactive plots (Plotly, Bokeh)
- Animated expression changes
- Network diagrams
- Custom color schemes

**Extra Features:**
- Shiny app to explore results
- Automatic report generation
- Support for different organisms
- Handle single-end or paired-end reads
- Batch correction options

**Performance:**
- Parallelize by chromosome
- Optimize memory usage
- Cache expensive steps
- Fast-mode option for quick results

---

## 📖 Useful Resources

**Tool Documentation:**
- FastQC: https://www.bioinformatics.babraham.ac.uk/projects/fastqc/
- STAR: https://github.com/alexdobin/STAR/blob/master/doc/STARmanual.pdf
- Salmon: https://salmon.readthedocs.io/
- DESeq2: https://bioconductor.org/packages/release/bioc/vignettes/DESeq2/inst/doc/DESeq2.html

**Tutorials:**
- RNA-seq analysis (Harvard): https://hbctraining.github.io/Intro-to-rnaseq-hpc-salmon/
- DESeq2 workflow: https://www.bioconductor.org/packages/devel/workflows/vignettes/rnaseqGene/inst/doc/rnaseqGene.html

**Example Pipelines:**
- nf-core/rnaseq: https://nf-co.re/rnaseq (production-ready, good reference)

**Containers:**
biocontainers/fastqc:v0.11.9_cv8
quay.io/biocontainers/star:2.7.10b--h9ee0642_0
quay.io/biocontainers/salmon:1.9.0--h7e5ed60_1
quay.io/biocontainers/subread:2.0.1--h5bf99c6_1
quay.io/biocontainers/bioconductor-deseq2:1.36.0--r41h9f5acd7_0

---

## 🤔 Questions to Consider

1. **How will you handle the sample metadata?**
   - Parse from filename?
   - Use a samplesheet CSV?
   - Pass as parameters?

2. **What quality thresholds will you use?**
   - Minimum read quality?
   - Minimum mapping rate?
   - Low-count gene filtering?

3. **How will you make results interpretable?**
   - Add gene names (not just IDs)?
   - Annotate with descriptions?
   - Highlight interesting genes?

4. **What if something fails?**
   - Retry strategies?
   - Skip optional steps?
   - Fail gracefully with informative errors?

5. **How will users run your pipeline?**
   - What parameters are configurable?
   - What are sensible defaults?
   - Clear documentation?

---

## 🎯 Success Criteria

**A successful pipeline:**
- ✅ Runs from start to finish without manual intervention
- ✅ Produces a list of differentially expressed genes
- ✅ Includes at least one meaningful visualization
- ✅ Is reproducible (same results every time)
- ✅ Has clear documentation explaining what it does
- ✅ Uses containers for all tools
- ✅ Handles errors gracefully

**Everything else is bonus!**

---

## 💭 Final Thoughts

This is **your** pipeline. There's no single "correct" solution. 

Some teams might focus on:
- 🚀 Speed and efficiency
- 📊 Beautiful visualizations  
- 🔬 Biological insights
- 🛠️ Robust error handling
- 📚 Excellent documentation
- 💡 Novel approaches

**All of these are valuable.**

The goal is to learn Nextflow, explore bioinformatics tools, and build something you're proud of.

**Have fun with it!** 🎉

---

## 📝 Tips

- Start simple: Get FastQC → STAR → counts working first
- Test with small data before running full dataset
- Use `.view()` to debug channels
- Don't worry about making it perfect
- Document your decisions
- Ask for help when stuck!

---

**Download the data and start building! Good luck! 🚀**