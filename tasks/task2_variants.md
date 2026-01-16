# Task 2: Variant Calling Pipeline ⭐⭐

**Difficulty:** Intermediate  
**Estimated Time:** 10-15 hours  
**Dataset Size:** ~1.2 GB

---

## 🎯 The Challenge

Build a germline variant calling pipeline that identifies genetic variants (SNPs and small indels) from whole genome sequencing data.

**In simple terms:** Find differences between your samples' DNA and the reference human genome.

---

## 📦 What You'll Get

**Dataset:**
- 3 whole genome sequencing samples (30x coverage)
- Human reference genome (chromosome 20 only)
- Known variant databases (dbSNP, gnomAD)
- High-confidence truth set for validation

**Download:** [LINK_TO_BE_PROVIDED]

---

## ✅ Minimum Requirements

Your pipeline must complete these core steps:

1. **Quality check** the sequencing data
2. **Align** reads to reference genome
3. **Call variants** (identify SNPs and indels)
4. **Filter variants** to remove low-quality calls
5. **Annotate variants** with functional information

That's the baseline. Everything else is up to you.

---

## 💡 Ideas & Inspiration

### Possible Approaches

**Alignment:**
- BWA-MEM (standard, reliable)
- BWA-MEM2 (faster version of BWA-MEM)
- Bowtie2 (alternative aligner)

**Variant Calling:**
- GATK HaplotypeCaller (industry standard)
- FreeBayes (simpler, faster)
- DeepVariant (uses deep learning)
- BCFtools mpileup (lightweight option)

**Filtering:**
- Hard filtering (set thresholds manually)
- VQSR (GATK's machine learning approach)
- Compare to known variants (dbSNP)
- Filter by coverage/quality

**Annotation:**
- VEP (Ensembl Variant Effect Predictor)
- SnpEff (fast, comprehensive)
- ANNOVAR (popular alternative)
- Add population frequencies (gnomAD)

### Interesting Directions

**Performance Focus:**
- Parallelize variant calling by genomic regions
- Compare runtimes across strategies
- Optimize memory usage
- Benchmark different callers

**Quality Focus:**
- Comprehensive QC at every step
- Validation against truth set
- Sensitivity/specificity analysis
- Filtering optimization

**Biological Focus:**
- Prioritize clinically relevant variants
- Flag known pathogenic variants
- Identify loss-of-function mutations
- Compare variants across samples

**Methodology:**
- Compare multiple variant callers
- Test different filtering strategies
- Ensemble calling (combine callers)
- Joint genotyping across samples

---

## 📊 Example Workflow

Here's **one possible approach**:
```
Raw FastQ files
    ↓
FastQC (quality control)
    ↓
BWA-MEM (align to reference)
    ↓
Mark duplicates (SAMtools)
    ↓
Base quality recalibration (GATK)
    ↓
HaplotypeCaller (call variants)
    ↓
Filter variants (GATK VariantFiltration)
    ↓
Annotate (VEP or SnpEff)
    ↓
Summary statistics
```

Alternative approaches:
- Skip BQSR if you want faster results
- Use FreeBayes instead of GATK
- Call variants per-region in parallel, then merge
- Add CNV detection
- Include structural variant calling

---

## 🎨 Make It Yours

### Stand Out With:

**Advanced Analysis:**
- Identify rare variants (< 1% population frequency)
- Flag compound heterozygous variants
- Predict variant impact on protein structure
- Identify variants in specific gene sets

**Parallelization:**
- Split genome into intervals
- Process intervals simultaneously
- Smart merging strategy
- Benchmark speedup

**Quality Metrics:**
- Concordance with truth set
- Transition/transversion ratio (Ti/Tv)
- Distribution of variant types
- Coverage analysis

**Visualization:**
- Variant quality score distributions
- Coverage plots
- Genomic context plots
- Interactive variant browser

**Extra Features:**
- Multi-sample joint calling
- Pedigree-aware calling (if you simulate family data)
- Somatic vs germline filtering
- Pharmocogenomics annotations

---

## 📖 Useful Resources

**Tool Documentation:**
- GATK Best Practices: https://gatk.broadinstitute.org/hc/en-us/articles/360035535932
- BWA: http://bio-bwa.sourceforge.net/bwa.shtml
- VEP: https://www.ensembl.org/info/docs/tools/vep/index.html
- SnpEff: https://pcingola.github.io/SnpEff/

**Tutorials:**
- GATK workflows: https://github.com/gatk-workflows
- Variant calling tutorial: https://hbctraining.github.io/variant_analysis/

**Example Pipelines:**
- nf-core/sarek: https://nf-co.re/sarek (cancer variant calling)

**Containers:**
```
biocontainers/bwa:0.7.17--h7132678_9
broadinstitute/gatk:4.3.0.0
quay.io/biocontainers/freebayes:1.3.6--hbfe0e7f_2
quay.io/biocontainers/ensembl-vep:107.0--pl5321h4a94de4_0
quay.io/biocontainers/snpeff:5.1--hdfd78af_2
```

---

## 🤔 Questions to Consider

1. **How will you handle parallelization?**
   - Process whole chromosome at once?
   - Split into smaller regions?
   - How to merge results?

2. **What quality filters make sense?**
   - Minimum coverage?
   - Quality score thresholds?
   - Allele frequency filters?

3. **How will you validate results?**
   - Compare to truth set?
   - Check known quality metrics?
   - Manual inspection of suspicious calls?

4. **What annotations are most useful?**
   - Gene names?
   - Predicted effects?
   - Population frequencies?
   - Clinical significance?

5. **How will you handle edge cases?**
   - Low coverage regions?
   - Failed processes?
   - Samples with unusual characteristics?

---

## 🔬 Scientific Considerations

**Key Quality Metrics:**
- **Ti/Tv ratio:** Should be ~2.0-2.1 for whole genome
- **Het/Hom ratio:** Expected ~1.5-2.0 for human
- **Known variants:** Should find most dbSNP sites in high-confidence regions
- **Coverage:** Mean 30x, but check distribution

**Common Pitfalls:**
- Low coverage regions produce false positives
- PCR duplicates inflate variant calls
- Alignment artifacts near indels
- Strand bias indicates artifacts

**Validation Strategy:**
- Use truth set (GIAB) to calculate precision/recall
- Check variant quality score distributions
- Inspect alignments for suspicious calls
- Compare to expected variant counts

---

## 🎯 Success Criteria

**A successful pipeline:**
- ✅ Identifies variants with reasonable accuracy
- ✅ Produces standard VCF output
- ✅ Annotates variants with functional information
- ✅ Includes quality metrics and statistics
- ✅ Runs efficiently (considers parallelization)
- ✅ Validates results against truth set (if possible)
- ✅ Documents approach and filtering decisions

**Bonus points for:**
- 🚀 Clever parallelization strategies
- 📊 Comprehensive quality analysis
- 🔬 Biological interpretation
- 💡 Novel filtering approaches
- 📈 Performance benchmarking

---

## 💭 Design Decisions

You'll need to make choices:

**Speed vs Accuracy:**
- GATK HaplotypeCaller: slower, very accurate
- FreeBayes: faster, good accuracy
- DeepVariant: slowest, highest accuracy

**Filtering Philosophy:**
- Conservative: fewer false positives, might miss real variants
- Permissive: catch more variants, more false positives
- Context-dependent: different filters for different variant types

**Parallelization Strategy:**
- Single large job: simple, slower
- Split by chromosome: moderate complexity
- Split into small intervals: complex, fastest

**All approaches are valid!** Document your choices.

---

## 🎨 Creative Extensions

Want to go beyond the basics?

- **Pharmacogenomics:** Flag variants affecting drug response
- **Pathogenicity prediction:** Integrate multiple prediction tools
- **Structural variants:** Add SV calling with tools like Manta
- **Visualization:** IGV-style genome browser views
- **Reporting:** Automated HTML report with key findings
- **Comparison:** Run multiple callers and create consensus calls
- **Trio analysis:** If you have family data, use it!

---

## 📝 Tips

- Start with a single sample before scaling to three
- Test on a small genomic region first (saves time!)
- Use truth set for validation - tells you if you're on the right track
- Check intermediate outputs (BAM files, VCF files)
- Monitor resource usage - variant calling can be memory-hungry
- Document your filtering criteria
- Don't reinvent GATK best practices (unless you want to!)

---

**This is more challenging than Task 1, but you've got this! 💪**

**Download the data and start building! 🧬**