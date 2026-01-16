# Nextflow Hackathon 2025

**Build a Reproducible Bioinformatics Pipeline in 24 Hours**

---

## 📅 Event Details

- **Seminar:** January 22, 2025 | 4:00 PM - 6:00 PM
- **Hackathon Start:** January 22, 2025 | 6:00 PM (tasks released)
- **Working Period:** 24 hours (remote work)
- **Presentations:** January 23, 2025 | 6:00 PM (in-person)
- **Location:** 
- **Max Teams:** 10 teams (2-4 people per team)

---

## 🎯 Challenge Overview

Build a complete bioinformatics analysis pipeline using Nextflow. Choose from three tasks of varying difficulty. Your pipeline should be reproducible, well-documented, and demonstrate Nextflow best practices.

**What is Nextflow?**
Nextflow is a workflow management system that enables scalable and reproducible scientific workflows using software containers.

---

## 📋 Tasks (Choose ONE)

### Task 1: RNA-Seq Analysis Pipeline ⭐ Beginner
**Goal:** Process RNA-seq data from raw reads to differential expression results.

**Required Steps:**
1. Quality control (FastQC + MultiQC)
2. Read alignment (STAR, HISAT2, or Salmon)
3. Quantification (HTSeq, featureCounts, or Salmon quant)
4. Differential expression (DESeq2 or edgeR)
5. Visualization (volcano plot + heatmap)

**Dataset:** 6 samples (3 control, 3 treatment), paired-end reads, human chr1 only
**Download:** [LINK TO BE PROVIDED]

---

### Task 2: Variant Calling Pipeline ⭐⭐ Intermediate
**Goal:** Identify germline variants from whole genome sequencing data.

**Required Steps:**
1. Quality control (FastQC + MultiQC)
2. Read alignment (BWA-MEM)
3. Variant calling (GATK HaplotypeCaller or FreeBayes)
4. Variant filtering (GATK VariantFiltration)
5. Variant annotation (VEP or SnpEff)

**Dataset:** 3 samples, 30x coverage, human chr20 only
**Download:** [LINK TO BE PROVIDED]

---

### Task 3: Multi-Omics Integration ⭐⭐⭐ Advanced
**Goal:** Integrate RNA-seq and DNA methylation data to identify correlations.

**Required Steps:**
1. RNA-seq processing (alignment + quantification)
2. Methylation analysis (Bismark alignment + methylation calling)
3. Data integration (correlation analysis)
4. Visualization (integrated plots showing methylation vs expression)
5. Biological interpretation (identify genes with correlated changes)

**Dataset:** 4 matched samples (RNA-seq + WGBS), human chr21 only
**Download:** [LINK TO BE PROVIDED]

---

## 💾 Datasets

All datasets are available for download at: **[GOOGLE DRIVE/ZENODO LINK]**

### Quick Download
```bash
# Download all data
wget [MAIN_DOWNLOAD_LINK] -O hackathon_data.tar.gz
tar -xzf hackathon_data.tar.gz

# Or download individual tasks
wget [TASK1_LINK] -O task1_rnaseq.tar.gz
wget [TASK2_LINK] -O task2_variants.tar.gz
wget [TASK3_LINK] -O task3_multiomics.tar.gz
```

### Dataset Sizes
- **Task 1:** ~800 MB
- **Task 2:** ~1.2 GB
- **Task 3:** ~1.5 GB

---

## ✅ Requirements

### Technical Requirements
- ✅ Use Nextflow DSL2
- ✅ All processes must use Docker/Singularity containers
- ✅ Create at least 2 configuration profiles (test, standard)
- ✅ Enable execution reports (timeline, trace, report)
- ✅ Implement error handling (retry strategies)
- ✅ Use parameters for all configurable options

### Deliverables
1. **GitHub Repository** (public)
   - Working Nextflow pipeline
   - Clear README with instructions
   - Example results
   - Workflow diagram

2. **Presentation** (7 min + 3 min Q&A)
   - Pipeline overview
   - Technical approach
   - Results
   - Challenges & solutions

---

## 📊 Scoring Rubric (100 points total)

| Category | Points | What We're Looking For |
|----------|--------|------------------------|
| **Code Quality & Documentation** | 25 | Clear code, good comments, excellent README |
| **Nextflow Best Practices** | 30 | Containers, configs, channels, modularity |
| **Scientific Validity** | 25 | Correct methods, quality results, innovation |
| **Presentation** | 20 | Clear explanation, good visuals, time management |
| **Bonus** | +10 | Outstanding innovation, extra features |

---

## 🚀 Getting Started

### 1. Install Prerequisites
```bash
# Install Nextflow
curl -s https://get.nextflow.io | bash
sudo mv nextflow /usr/local/bin/

# Verify installation
nextflow -version

# Install Docker (or Singularity)
# macOS: Download Docker Desktop
# Linux: sudo apt-get install docker.io
# Windows: Download Docker Desktop (use WSL2)
```

### 2. Test Your Setup
```bash
# Test Nextflow
nextflow run hello

# Test Docker
docker run hello-world

# Pull a test bioinformatics container
docker pull biocontainers/fastqc:v0.11.9_cv8
```

### 3. Download Data
```bash
# Download your chosen task dataset
wget [TASK_LINK] -O task_data.tar.gz
tar -xzf task_data.tar.gz
```

### 4. Create Your Repository
```bash
# Fork the template repository or create new
git clone https://github.com/[your-username]/[your-repo].git
cd [your-repo]

# Create basic structure
mkdir -p modules bin conf
touch main.nf nextflow.config README.md
```

---

## 📁 Repository Structure

```
your-pipeline/
├── main.nf                 # Main workflow
├── nextflow.config         # Configuration
├── modules/                # Process modules
│   ├── fastqc.nf
│   ├── alignment.nf
│   └── ...
├── bin/                    # Custom scripts
│   └── create_plots.R
├── conf/                   # Additional configs
├── README.md              # Documentation
├── LICENSE
└── .gitignore
```

---

## 📖 Example: Minimal Pipeline

Here's a simple example to get you started:

```groovy
#!/usr/bin/env nextflow
nextflow.enable.dsl=2

// Parameters
params.input = 'samplesheet.csv'
params.outdir = 'results'

// Process
process FASTQC {
    tag "$sample_id"
    publishDir "${params.outdir}/fastqc", mode: 'copy'
    container 'biocontainers/fastqc:v0.11.9_cv8'
    
    input:
    tuple val(sample_id), path(reads)
    
    output:
    path "*.{html,zip}"
    
    script:
    """
    fastqc -q ${reads}
    """
}

// Workflow
workflow {
    Channel
        .fromPath(params.input)
        .splitCsv(header: true)
        .map { row -> tuple(row.sample, [file(row.fastq_1), file(row.fastq_2)]) }
        .set { samples_ch }
    
    FASTQC(samples_ch)
}
```

---

## 🛠️ Useful Resources

### Nextflow
- **Documentation:** https://nextflow.io/docs/latest/
- **Training:** https://training.nextflow.io/
- **nf-core Pipelines:** https://nf-co.re/ (great examples!)
- **Patterns:** https://nextflow-io.github.io/patterns/

### Containers
- **BioContainers:** https://biocontainers.pro/
- **Quay.io:** https://quay.io/organization/biocontainers

### Tools Documentation
- **FastQC:** https://www.bioinformatics.babraham.ac.uk/projects/fastqc/
- **STAR:** https://github.com/alexdobin/STAR
- **GATK:** https://gatk.broadinstitute.org/
- **DESeq2:** https://bioconductor.org/packages/DESeq2/

---

## 💡 Tips for Success

### Strategy
1. **Start simple** - Get end-to-end working first
2. **Test frequently** - Use `-resume` to save time
3. **Modularize early** - Separate processes into modules
4. **Document as you go** - Don't wait until the end

### Common Pitfalls to Avoid
- ❌ Hardcoding paths
- ❌ Using `:latest` tags
- ❌ No error handling
- ❌ Poor documentation
- ❌ Committing large files to Git

### Debugging
```bash
# Find failed task directory
ls -la work/*/*

# Check the command
cat work/xx/xxxxxx/.command.sh

# Check logs
cat work/xx/xxxxxx/.command.log
cat work/xx/xxxxxx/.command.err

# Resume after fixing
nextflow run main.nf -resume
```

---

## 🎨 Beyond the Requirements: Creative Ideas

The tasks above define **minimum requirements**. Here are ideas to make your pipeline stand out:

### Pipeline Enhancements
- 🔄 **Multi-step QC:** Add adapter trimming, rRNA removal
- ⚡ **Optimization:** Implement parallel processing strategies
- 📊 **Rich Reports:** Create interactive HTML reports with plots
- 🧪 **Validation:** Include statistical validation of results
- 🔧 **Flexibility:** Support multiple tools (e.g., STAR or HISAT2)

### Analysis Extensions

**For RNA-Seq (Task 1):**
- Gene Ontology (GO) enrichment analysis
- KEGG pathway analysis
- Gene Set Enrichment Analysis (GSEA)
- Alternative splicing analysis
- Fusion gene detection
- Interactive Shiny app for results exploration

**For Variant Calling (Task 2):**
- Variant quality score recalibration (VQSR)
- CNV detection
- Structural variant calling
- Population frequency annotation (gnomAD)
- Clinical significance (ClinVar)
- Pharmocogenomics annotations

**For Multi-Omics (Task 3):**
- Network analysis (gene co-expression networks)
- Clustering analysis
- Machine learning predictions
- Integration with other data types (proteomics, metabolomics)
- Causal inference analysis

### Visualization Ideas
- 📈 Interactive plots (Plotly, Bokeh)
- 🌐 Web dashboard (Shiny, Dash)
- 🎞️ Animated visualizations
- 🗺️ Genomic browser tracks (IGV)
- 📊 Multi-panel figures

### Technical Innovations
- ☁️ **Cloud-ready:** AWS Batch, Google Cloud support
- 📦 **Packaging:** Conda environment files
- 🔍 **Testing:** Automated testing with pytest or testthat
- 📝 **Logging:** Comprehensive logging system
- 🎯 **Benchmarking:** Performance comparisons

### Documentation Excellence
- 📹 Video tutorial showing how to run your pipeline
- 📚 Detailed methods section (publication-ready)
- 🖼️ Architecture diagrams
- 💬 Usage examples for different scenarios
- ❓ FAQ section

### Community Contribution
- 🌟 Create reusable nf-core style modules
- 📦 Package for easy installation
- 🐛 Include test datasets
- 🤝 Make it easy for others to contribute

---

## 🏆 Judging Criteria Details

### Code Quality (25 points)
- Clean, readable code with consistent style
- Comprehensive comments explaining logic
- Modular design (separate processes/modules)
- Proper error handling and edge cases
- Well-structured repository

### Nextflow Best Practices (30 points)
- All processes containerized (Docker/Singularity)
- Multiple configuration profiles
- Efficient channel operations
- Resource labels and dynamic allocation
- Process directives (publishDir, tag, etc.)
- Execution reports enabled

### Scientific Validity (25 points)
- Biologically/scientifically sound approach
- Appropriate tools and parameters
- Quality control at each step
- Results interpretation
- Innovation and creativity

### Presentation (20 points)
- Clear and engaging explanation
- Good visual aids (diagrams, plots)
- Demonstrates understanding
- Answers questions well
- Time management

### Bonus Points (+10)
- Outstanding innovation or novel approach
- Exceptional documentation
- Extra features beyond requirements
- Community contribution potential

---

## 📅 Timeline

### Before the Event
- **Now - Jan 21:** Set up your environment, study Nextflow basics
- **Jan 21:** Download datasets, test your setup

### Event Day 1
- **Jan 22, 4:00 PM:** Seminar begins (attend in-person or via Zoom)
- **Jan 22, 6:00 PM:** Tasks released, hackathon begins
- **Jan 22, 6:00 PM - Jan 23, 6:00 PM:** 24-hour working period

### Event Day 2
- **Jan 23, 6:00 PM:** Submission deadline (repository frozen)
- **Jan 23, 6:00-6:30 PM:** Setup and judge review
- **Jan 23, 6:30-9:00 PM:** Team presentations
- **Jan 23, 9:00-9:30 PM:** Awards ceremony

---

## 📝 Submission Instructions

### What to Submit
1. **GitHub Repository URL**
2. **Team Information** (names, emails)
3. **Task Chosen** (1, 2, or 3)

### Submission Form
Submit via: **[GOOGLE FORM LINK]**

### Deadline
**January 23, 2025, 6:00 PM EST (sharp!)**

### What Should Be in Your Repository
- ✅ Complete, working Nextflow pipeline
- ✅ README.md with clear instructions
- ✅ Example outputs (small files only!)
- ✅ Nextflow execution reports
- ✅ Presentation slides (PDF)

### What NOT to Commit
- ❌ Large data files (use .gitignore)
- ❌ Work directories
- ❌ Intermediate results
- ❌ Personal credentials

---

## 🤝 Support During Hackathon

### Get Help
- **Slack:** [SLACK WORKSPACE LINK]
- **Email:** [ORGANIZER EMAIL]
- **Office Hours:** Jan 22-23, available on Slack

### Communication Guidelines
- Ask questions in public Slack channels
- Help other teams when you can
- Share resources, not solutions
- Be respectful and supportive

---

## 🎁 Prizes

- **1st Place:** [PRIZE]
- **2nd Place:** [PRIZE]
- **3rd Place:** [PRIZE]
- **Best Visualization:** [PRIZE]
- **Most Innovative:** [PRIZE]

---

## ❓ FAQ

**Q: Can I work alone or must I have a team?**
A: Teams of 2-4 are recommended, but solo participants are welcome.

**Q: Can I use existing nf-core modules?**
A: Yes, but you must understand them and give proper credit.

**Q: What if I've never used Nextflow before?**
A: Perfect! This is a learning opportunity. Attend the seminar and use the resources.

**Q: Can I use tools not mentioned in the task description?**
A: Absolutely! Feel free to use any appropriate tools.

**Q: What if my pipeline doesn't finish in 24 hours?**
A: Submit what you have. Partial working solutions with good documentation can still win.

**Q: Can I get help during the hackathon?**
A: Yes! Use Slack for questions. We encourage collaboration and learning.

**Q: Do I need access to HPC or cloud resources?**
A: No. All tasks are designed to run on a laptop with 8GB+ RAM.

---

## 📜 Rules

1. All work must be done during the 24-hour period
2. Open source tools only
3. Repository must be public by submission deadline
4. Proper attribution for any code/resources used
5. Be respectful and supportive of other teams
6. Have fun and learn!

---

## 🙏 Acknowledgments

Special thanks to:
- Nextflow community
- nf-core project
- Bioconda and BioContainers
- Graduate Student Government (GSG) at Northeastern

---

## 📞 Contact

**Organizers:**
- [Name 1] - [email1]
- [Name 2] - [email2]

**Event Website:** [WEBSITE]
**Slack Workspace:** [SLACK LINK]

---

**Good luck and happy pipelining! 🚀🧬**

---

*Last updated: January 15, 2025*
