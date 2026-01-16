# Task 3: Multi-Omics Integration ⭐⭐⭐

**Difficulty:** Advanced  
**Estimated Time:** 12-20 hours  
**Dataset Size:** ~1.5 GB

---

## 🎯 The Challenge

Build a pipeline that integrates RNA-seq and DNA methylation data to discover relationships between gene expression and epigenetic regulation.

**In simple terms:** Find genes where methylation changes correlate with expression changes - potential regulatory insights!

---

## 📦 What You'll Get

**Dataset:**
- 4 matched samples (each has RNA-seq AND methylation data)
- 2 healthy controls + 2 disease samples
- RNA-seq: paired-end reads
- WGBS (whole genome bisulfite sequencing): methylation data
- Reference genome and annotations (chromosome 21 only)

**Download:** [LINK_TO_BE_PROVIDED]

---

## ✅ Minimum Requirements

Your pipeline must:

1. **Process RNA-seq data** → gene expression values
2. **Process methylation data** → CpG methylation levels
3. **Integrate the two data types** → find correlations
4. **Visualize relationships** → show the integration
5. **Interpret results** → biological insights

The integration is the key challenge here.

---

## 💡 Ideas & Inspiration

### RNA-Seq Processing

**Standard approach:**
- QC with FastQC
- Align with STAR/HISAT2
- Quantify with HTSeq/featureCounts
- Normalize expression values

**Alternative:**
- Use Salmon for fast quantification
- Consider TPM vs FPKM vs raw counts
- Filter lowly expressed genes

### Methylation Processing

**Tools:**
- Bismark (standard for WGBS)
- bwa-meth (alternative)
- BSMAP (another option)

**Key steps:**
- Align bisulfite-converted reads
- Call methylation at CpG sites
- Calculate methylation percentage (β-values)
- Identify differentially methylated regions (DMRs)

### Integration Strategies

**Correlation Analysis:**
- Match genes to nearby CpG sites
- Calculate expression-methylation correlation
- Identify negative correlations (typical for promoters)

**Differential Analysis:**
- Find differentially expressed genes
- Find differentially methylated regions
- Overlap the two lists

**Mechanistic:**
- Focus on promoter regions (expect negative correlation)
- Look at gene bodies (might see positive correlation)
- Enhancer regions (more complex)

**Statistical:**
- Multiple testing correction
- Permutation testing
- Integrative clustering

---

## 📊 Example Workflow

Here's **one way** to approach it:
```
RNA-seq Processing:
FastQ → QC → Align → Quantify → Normalized expression matrix

Methylation Processing:
FastQ → QC → Bismark align → Extract methylation → β-value matrix

Integration:
1. Define regions of interest (promoters, gene bodies)
2. Match genes to methylation sites
3. Calculate correlations
4. Statistical testing
5. Biological interpretation

Visualization:
- Scatter plots (expression vs methylation)
- Heatmaps (integrated data)
- Genome browser tracks
- Network diagrams
```

But you could also:
- Use dimensionality reduction (PCA, UMAP)
- Cluster by patterns
- Machine learning for prediction
- Pathway-level analysis
- Time series if you add that complexity

---

## 🎨 Creative Approaches

### Analysis Directions

**Discovery-focused:**
- Which genes show strong methylation-expression relationships?
- Are there clusters of co-regulated genes?
- Identify novel regulatory relationships

**Hypothesis-driven:**
- Do tumor suppressors show promoter hypermethylation?
- Are developmental genes affected?
- Tissue-specific patterns?

**Methods comparison:**
- Different correlation methods
- Various region definitions
- Alternative integration strategies

**Visualization-heavy:**
- Interactive exploration tools
- Integrated genome browser
- Network diagrams showing relationships
- Publication-quality figures

### Stand Out Features

**Statistical Rigor:**
- Multiple testing correction
- Permutation-based p-values
- Bootstrap confidence intervals
- Power analysis

**Biological Interpretation:**
- GO enrichment on correlated genes
- Pathway analysis
- Transcription factor binding site enrichment
- Literature mining

**Technical Excellence:**
- Efficient processing of both data types
- Memory-optimized integration
- Modular, reusable components
- Comprehensive QC

**Innovation:**
- Machine learning integration
- Network analysis
- Novel visualization approaches
- Predictive modeling

---

## 📖 Useful Resources

**RNA-seq:**
- See Task 1 resources

**Methylation:**
- Bismark: https://github.com/FelixKrueger/Bismark
- Bismark User Guide: https://rawgit.com/FelixKrueger/Bismark/master/Docs/Bismark_User_Guide.html
- WGBS analysis: https://www.bioconductor.org/help/course-materials/2015/BioC2015/methylation450k.html

**Integration:**
- No single "standard" tool - this is where you innovate!
- Consider: correlation analysis, regression, clustering
- R packages: methylKit, bsseq, minfi
- Python: scipy, statsmodels, scikit-learn

**Example Pipelines:**
- nf-core/methylseq: https://nf-co.re/methylseq

**Containers:**
```
# RNA-seq (see Task 1)
quay.io/biocontainers/star:2.7.10b--h9ee0642_0
quay.io/biocontainers/salmon:1.9.0--h7e5ed60_1

# Methylation
quay.io/biocontainers/bismark:0.24.0--hdfd78af_0
quay.io/biocontainers/bwameth:0.2.2--py_1

# Analysis
rocker/tidyverse:4.2.2  # R with tidyverse
jupyter/scipy-notebook  # Python scientific stack
```

---

## 🤔 Deep Questions

1. **How do you define "nearby" CpG sites?**
   - Within promoter (how far upstream)?
   - Closest CpG to TSS?
   - Average across a region?
   - Specific CpG islands?

2. **What correlation metric?**
   - Pearson (linear relationship)?
   - Spearman (monotonic)?
   - Kendall (rank-based)?
   - Mutual information?

3. **How do you handle multiple CpGs per gene?**
   - Average methylation?
   - Maximum correlation?
   - Region-based approach?
   - Model all sites?

4. **What about multiple testing?**
   - Bonferroni (conservative)?
   - FDR (Benjamini-Hochberg)?
   - Permutation-based?
   - q-values?

5. **How do you validate findings?**
   - Literature comparison?
   - Known biology?
   - Independent dataset?
   - Experimental follow-up suggestions?

---

## 🔬 Scientific Considerations

**Expected Biology:**
- **Promoter methylation** usually anti-correlates with expression
- **Gene body methylation** might positively correlate
- **Tissue-specific genes** may show strong patterns
- **Housekeeping genes** typically unmethylated

**Technical Challenges:**
- Bisulfite conversion can damage DNA
- Coverage requirements differ between data types
- Normalization matters
- Batch effects can confound

**Statistical Issues:**
- Correlation ≠ causation
- Multiple testing with thousands of genes
- Sample size limitations
- Technical vs biological variation

---

## 🎯 Success Criteria

**A successful pipeline:**
- ✅ Processes both RNA-seq and methylation data correctly
- ✅ Integrates the data in a meaningful way
- ✅ Identifies candidate genes with correlated changes
- ✅ Visualizes the integration clearly
- ✅ Provides biological interpretation
- ✅ Documents methodology thoroughly
- ✅ Handles edge cases gracefully

**Excellence indicators:**
- 🧬 Novel biological insights
- 📊 Sophisticated statistical analysis
- 🎨 Beautiful, informative visualizations
- 💻 Elegant code architecture
- 📝 Thorough documentation
- 🔬 Validation of findings

---

## 🎨 Integration Examples

### Simple Correlation
```r
# For each gene:
# 1. Get expression values (4 samples)
# 2. Get methylation values (4 samples)
# 3. Calculate correlation
# 4. Test significance
```

### Region-Based
```python
# Define promoter regions (-2kb to +500bp of TSS)
# Average methylation across region
# Correlate with gene expression
# Identify significantly correlated pairs
```

### Differential Approach
```r
# Find DE genes (control vs disease)
# Find DMRs (control vs disease)
# Overlap gene-linked regions
# Focus on genes with coordinated changes
```

### Machine Learning
```python
# Use methylation to predict expression
# Feature importance analysis
# Identify key regulatory regions
# Cross-validation
```

---

## 💭 Creative Extensions

**Advanced Analysis:**
- Include TF binding site predictions
- Add histone modification data (simulated)
- 3D genome organization context
- Allele-specific analysis

**Methods Development:**
- Compare integration approaches
- New visualization types
- Novel statistical tests
- Improved correlation metrics

**Biological Focus:**
- Disease-relevant pathways
- Druggable targets
- Biomarker discovery
- Mechanistic models

**Software Engineering:**
- Create reusable modules
- Support multiple organisms
- Scalable to full genomes
- User-friendly interface

---

## 📝 Pro Tips

- **Start with one sample pair** - get the integration working first
- **Visualize early and often** - plots reveal issues
- **Use known biology** - validate your approach on well-studied genes
- **Document decisions** - why did you choose that region size?
- **Handle missing data** - not all genes have nearby methylation data
- **Mind the multiple testing** - many tests = need correction
- **Focus on interpretability** - cool analysis means nothing if you can't explain it

---

## 🧠 This Task is About...

- **Integration skills** - combining different data types
- **Statistical thinking** - proper correlation and testing
- **Biological interpretation** - what do the results mean?
- **Pipeline design** - handling complex workflows
- **Communication** - explaining your approach clearly

It's okay if you don't finish everything. Focus on doing **one thing really well** rather than everything superficially.

---

## 🌟 Remember

This is an **open-ended** challenge. There's no single right answer.

Some teams might excel at:
- 🔬 Biological insights
- 📊 Statistical rigor
- 💻 Technical implementation
- 🎨 Visualization quality
- 📝 Clear communication

**All approaches are valuable.**

The goal is to learn, explore, and build something interesting.

---

**This is the most challenging task - embrace it! 🚀**

**Download the data and start experimenting! 🧬🔬**