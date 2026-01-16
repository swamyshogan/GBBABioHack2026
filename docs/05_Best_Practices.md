# Nextflow Best Practices

Write clean, maintainable, and production-ready Nextflow pipelines.

---

## 📖 Table of Contents

1. [Code Organization](#code-organization)
2. [Naming Conventions](#naming-conventions)
3. [Documentation](#documentation)
4. [Error Handling](#error-handling)
5. [Resource Management](#resource-management)
6. [Security & Safety](#security--safety)
7. [Testing](#testing)
8. [Version Control](#version-control)

---

## Code Organization

### Project Structure
````
my-pipeline/
├── main.nf                     # Main workflow entry point
├── nextflow.config             # Main configuration
├── modules/                    # Reusable process modules
│   ├── fastqc.nf
│   ├── trimming.nf
│   ├── alignment.nf
│   └── quantification.nf
├── subworkflows/              # Composed workflows
│   ├── qc_workflow.nf
│   └── alignment_workflow.nf
├── bin/                       # Custom scripts
│   ├── create_count_matrix.py
│   └── run_deseq2.R
├── conf/                      # Additional configs
│   ├── base.config
│   ├── resources.config
│   └── modules.config
├── lib/                       # Groovy helper functions
│   └── Utils.groovy
├── assets/                    # Static files
│   └── multiqc_config.yaml
├── docs/                      # Documentation
│   ├── usage.md
│   └── output.md
├── test_data/                 # Small test dataset
│   └── samplesheet.csv
├── .github/                   # CI/CD
│   └── workflows/
│       └── ci.yml
├── README.md                  # Main documentation
├── CHANGELOG.md              # Version history
├── LICENSE                    # License file
└── .gitignore                # Git ignore patterns
````

### Separating Concerns

**❌ Bad: Everything in one file**
````groovy
// main.nf - 1000 lines of code
process FASTQC { ... }
process TRIMMING { ... }
process ALIGNMENT { ... }
// ... 20 more processes
workflow { ... }
````

**✅ Good: Modular organization**
````groovy
// main.nf
include { QC_WORKFLOW } from './subworkflows/qc_workflow'
include { ALIGNMENT_WORKFLOW } from './subworkflows/alignment_workflow'
include { QUANTIFICATION_WORKFLOW } from './subworkflows/quantification_workflow'

workflow {
    reads_ch = Channel.fromPath(params.input).splitCsv(header: true)
    
    QC_WORKFLOW(reads_ch)
    ALIGNMENT_WORKFLOW(QC_WORKFLOW.out.trimmed)
    QUANTIFICATION_WORKFLOW(ALIGNMENT_WORKFLOW.out.bam)
}
````

### Module Template

**modules/fastqc.nf:**
````groovy
/*
 * FastQC quality control
 * 
 * Description:
 *   Performs quality control checks on raw sequence data
 * 
 * Input:
 *   tuple val(meta), path(reads) - Sample metadata and FastQ files
 * 
 * Output:
 *   path("*.{html,zip}") - FastQC reports
 * 
 * Container:
 *   biocontainers/fastqc:v0.11.9_cv8
 */

process FASTQC {
    tag "$meta.id"
    label 'process_low'
    publishDir "${params.outdir}/fastqc", mode: 'copy'
    container 'biocontainers/fastqc:v0.11.9_cv8'
    
    input:
    tuple val(meta), path(reads)
    
    output:
    tuple val(meta), path("*.{html,zip}"), emit: reports
    path "versions.yml"                  , emit: versions
    
    script:
    def prefix = meta.id
    """
    fastqc -q -t ${task.cpus} ${reads}
    
    cat <<-END_VERSIONS > versions.yml
    "${task.process}":
        fastqc: \$(fastqc --version | sed 's/FastQC v//')
    END_VERSIONS
    """
}
````

---

## Naming Conventions

### Process Names
````groovy
// ✅ Use UPPER_CASE for processes
process FASTQC { }
process STAR_ALIGN { }
process DESEQ2_DIFFERENTIAL_EXPRESSION { }

// ❌ Avoid lowercase or mixed case
process fastqc { }              // Bad
process starAlign { }           // Bad
process DESeq2Analysis { }      // Bad
````

### Variable Names
````groovy
// ✅ Use snake_case for variables and channels
def sample_id = "sample1"
def input_files = Channel.fromPath("*.fastq")
reads_ch = Channel.fromPath(params.reads)

// ❌ Avoid camelCase in Nextflow (save for Groovy methods)
def sampleId = "sample1"        // Less consistent with Nextflow style
````

### Parameter Names
````groovy
// ✅ Clear, descriptive parameter names
params.input = 'samplesheet.csv'
params.outdir = 'results'
params.genome = 'hg38'
params.skip_trimming = false
params.max_memory = '128.GB'

// ❌ Avoid abbreviations or unclear names
params.i = 'samplesheet.csv'    // What is 'i'?
params.out = 'results'          // Unclear
params.mem = '128.GB'           // Abbreviation
````

### Channel Names
````groovy
// ✅ Descriptive channel names ending in _ch
reads_ch = Channel.fromPath("*.fastq")
trimmed_ch = TRIMMING.out
aligned_bam_ch = ALIGNMENT.out.bam

// ✅ For complex data, describe content
sample_metadata_ch = Channel.fromPath('metadata.csv')
rnaseq_counts_ch = HTSEQ.out.counts
````

---

## Documentation

### README.md Template
````markdown
# Pipeline Name

Brief description of what the pipeline does.

## Quick Start
```bash
nextflow run main.nf --input samplesheet.csv --outdir results
```

## Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `--input` | file | - | Path to input samplesheet (required) |
| `--outdir` | path | `results` | Output directory |
| `--genome` | string | `hg38` | Reference genome |
| `--skip_qc` | flag | `false` | Skip quality control steps |

## Input Format

### Samplesheet CSV
```csv
sample,fastq_1,fastq_2,condition
sample1,data/sample1_R1.fastq.gz,data/sample1_R2.fastq.gz,control
sample2,data/sample2_R1.fastq.gz,data/sample2_R2.fastq.gz,treatment
```

## Output Structure
````
results/
├── fastqc/           # Quality control reports
├── trimmed/          # Trimmed reads
├── aligned/          # BAM files
├── counts/           # Gene counts
└── differential/     # DE analysis results
````

## Requirements

- Nextflow >= 23.10.0
- Docker or Singularity
- 8 GB RAM minimum

## Citation

If you use this pipeline, please cite: [CITATION]
````

### Process Documentation
````groovy
/**
 * STAR alignment process
 * 
 * Aligns RNA-seq reads to reference genome using STAR aligner
 * 
 * @param meta Map containing sample metadata (id, condition, etc.)
 * @param reads List of FastQ files [R1, R2] for paired-end
 * @param index Path to STAR genome index directory
 * @return Tuple of (meta, aligned BAM file, alignment stats)
 * 
 * @example
 * STAR_ALIGN(sample_ch, star_index_ch)
 */
process STAR_ALIGN {
    // Process definition
}
````

### Inline Comments
````groovy
workflow {
    // Load samples from CSV and create channel
    // Format: [meta, [fastq_1, fastq_2]]
    samples_ch = Channel
        .fromPath(params.input, checkIfExists: true)
        .splitCsv(header: true)
        .map { row -> 
            def meta = [
                id: row.sample,
                condition: row.condition,
                single_end: false
            ]
            tuple(meta, [file(row.fastq_1), file(row.fastq_2)])
        }
    
    // Quality control on raw reads
    FASTQC(samples_ch)
    
    // Trimming adapters - skip if --skip_trimming is set
    if (!params.skip_trimming) {
        TRIMMOMATIC(samples_ch)
        trimmed_ch = TRIMMOMATIC.out.trimmed
    } else {
        trimmed_ch = samples_ch
    }
    
    // Alignment using STAR
    STAR_ALIGN(trimmed_ch, star_index)
}
````

---

## Error Handling

### Input Validation
````groovy
// Validate parameters at the start
def validateParams() {
    def errors = []
    
    // Check required parameters
    if (!params.input) {
        errors << "Missing required parameter: --input"
    }
    
    // Check file exists
    if (params.input && !file(params.input).exists()) {
        errors << "Input file does not exist: ${params.input}"
    }
    
    // Check valid values
    if (params.genome !in ['hg38', 'hg19', 'mm10', 'mm39']) {
        errors << "Invalid genome: ${params.genome}. Must be one of: hg38, hg19, mm10, mm39"
    }
    
    // Check numeric ranges
    if (params.min_coverage < 0) {
        errors << "min_coverage must be >= 0, got: ${params.min_coverage}"
    }
    
    // Report all errors
    if (errors) {
        error "Parameter validation failed:\n  " + errors.join("\n  ")
    }
}

// Call at start of workflow
validateParams()
````

### Graceful Error Messages
````groovy
process ALIGNMENT {
    errorStrategy { task.exitStatus in 137..140 ? 'retry' : 'finish' }
    maxRetries 3
    
    input:
    tuple val(meta), path(reads)
    
    script:
    """
    # Check input files exist and are not empty
    if [ ! -s ${reads[0]} ]; then
        echo "ERROR: Input file ${reads[0]} is empty or missing"
        exit 1
    fi
    
    # Run alignment with error checking
    bwa mem reference.fa ${reads[0]} ${reads[1]} > ${meta.id}.sam || {
        echo "ERROR: BWA alignment failed for sample ${meta.id}"
        echo "Check that your reads are properly formatted"
        exit 1
    }
    """
}
````

### Retry Strategies
````groovy
process MEMORY_INTENSIVE {
    // Increase memory on retry
    memory { 8.GB * task.attempt }
    time { 2.h * task.attempt }
    
    // Retry memory errors, fail on others
    errorStrategy { 
        task.exitStatus in 137..140 ? 'retry' : 
        task.exitStatus == 143 ? 'retry' :
        'terminate' 
    }
    maxRetries 3
    
    script:
    """
    echo "Attempt ${task.attempt} with ${task.memory} memory"
    
    # Your memory-intensive command
    big_analysis.sh input.txt
    """
}
````

---

## Resource Management

### Resource Labels
````groovy
// conf/base.config
process {
    // Defaults
    cpus = 1
    memory = 2.GB
    time = 1.h
    
    // Labels for common resource patterns
    withLabel: 'process_low' {
        cpus = 2
        memory = 4.GB
        time = 1.h
    }
    
    withLabel: 'process_medium' {
        cpus = 6
        memory = 16.GB
        time = 4.h
    }
    
    withLabel: 'process_high' {
        cpus = 12
        memory = 64.GB
        time = 8.h
    }
    
    withLabel: 'process_long' {
        time = 24.h
    }
    
    // Specific process overrides
    withName: 'STAR_ALIGN' {
        cpus = 16
        memory = 64.GB
        time = 6.h
    }
}
````

**Using labels:**
````groovy
process FASTQC {
    label 'process_low'
    
    script:
    """
    fastqc -t ${task.cpus} input.fastq
    """
}

process STAR_ALIGN {
    label 'process_high'
    
    script:
    """
    STAR --runThreadN ${task.cpus} ...
    """
}
````

### Dynamic Resources
````groovy
process SORT_BAM {
    cpus 4
    
    // Scale memory with input size
    memory { 
        def size_gb = bam.size() / 1000000000
        4.GB * Math.max(1, Math.ceil(size_gb / 2))
    }
    
    input:
    path bam
    
    script:
    """
    samtools sort -@ ${task.cpus} \\
        -m ${task.memory.toGiga() / task.cpus}G \\
        ${bam}
    """
}
````

### Resource Limits
````groovy
// nextflow.config
params {
    max_cpus = 16
    max_memory = 128.GB
    max_time = 240.h
}

process {
    cpus = { check_max(task.cpus, params.max_cpus) }
    memory = { check_max(task.memory, params.max_memory) }
    time = { check_max(task.time, params.max_time) }
}

// Helper function
def check_max(obj, max) {
    if (obj instanceof nextflow.util.MemoryUnit && obj.compareTo(max as nextflow.util.MemoryUnit) == 1) {
        return max as nextflow.util.MemoryUnit
    }
    if (obj instanceof nextflow.util.Duration && obj.compareTo(max as nextflow.util.Duration) == 1) {
        return max as nextflow.util.Duration
    }
    if (obj instanceof Integer && obj > max) {
        return max
    }
    return obj
}
````

---

## Security & Safety

### Never Hardcode Sensitive Data
````groovy
// ❌ BAD: Hardcoded credentials
process DOWNLOAD {
    script:
    """
    wget --user=myuser --password=mypass https://data.example.com/file.txt
    """
}

// ✅ GOOD: Use environment variables or secrets
process DOWNLOAD {
    secret 'MY_PASSWORD'
    
    script:
    """
    wget --user=${params.username} --password=\$MY_PASSWORD https://data.example.com/file.txt
    """
}
````

### Validate External Inputs
````groovy
// ❌ BAD: Trusting user input directly
process ANALYZE {
    script:
    """
    analyze_tool --param ${params.user_input}
    """
}

// ✅ GOOD: Sanitize and validate
def validateInput(input) {
    // Only allow alphanumeric and underscores
    if (!input.matches(/^[a-zA-Z0-9_]+$/)) {
        error "Invalid input: ${input}. Only alphanumeric characters and underscores allowed."
    }
    return input
}

process ANALYZE {
    script:
    def safe_input = validateInput(params.user_input)
    """
    analyze_tool --param ${safe_input}
    """
}
````

### File Existence Checks
````groovy
// ✅ Always check files exist
Channel
    .fromPath(params.input, checkIfExists: true)
    .ifEmpty { error "No files found matching: ${params.input}" }
    .set { input_ch }

// ✅ Check before using in process
process ANALYZE {
    input:
    path input_file
    
    script:
    """
    if [ ! -f ${input_file} ]; then
        echo "ERROR: Input file not found: ${input_file}"
        exit 1
    fi
    
    analyze.sh ${input_file}
    """
}
````

---

## Testing

### Unit Testing Processes

**test/test_fastqc.nf:**
````groovy
#!/usr/bin/env nextflow
nextflow.enable.dsl=2

include { FASTQC } from '../modules/fastqc'

workflow test_fastqc {
    input = [
        [id: 'test_sample'],
        file("${baseDir}/test_data/test.fastq.gz")
    ]
    
    FASTQC(Channel.of(input))
}

workflow {
    test_fastqc()
}
````

**Run test:**
````bash
nextflow run test/test_fastqc.nf
````

### Integration Testing

**test/test_pipeline.nf:**
````groovy
#!/usr/bin/env nextflow
nextflow.enable.dsl=2

params.input = "${baseDir}/test_data/samplesheet.csv"
params.outdir = "test_results"

include { FULL_WORKFLOW } from '../main'

workflow {
    FULL_WORKFLOW()
}
````

### Continuous Integration

**.github/workflows/ci.yml:**
````yaml
name: Nextflow CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Install Nextflow
        run: |
          wget -qO- https://get.nextflow.io | bash
          sudo mv nextflow /usr/local/bin/
          
      - name: Run pipeline test
        run: |
          nextflow run test/test_pipeline.nf -profile test,docker
````

---

## Version Control

### .gitignore Template
````gitignore
# Nextflow
work/
.nextflow/
.nextflow.*
*.log

# Results
results/
output/

# Data (don't commit large files)
*.fastq
*.fastq.gz
*.fq.gz
*.bam
*.sam
*.vcf
*.bcf
*.fa
*.fasta

# Python
__pycache__/
*.py[cod]
.ipynb_checkpoints/

# R
.Rhistory
.RData
.Rproj.user/

# OS
.DS_Store
Thumbs.db
*.swp
*.swo
*~

# IDE
.vscode/
.idea/
*.sublime-*

# Temporary
tmp/
temp/
*.tmp
````

### Commit Messages
````bash
# ✅ Good commit messages
git commit -m "Add STAR alignment module"
git commit -m "Fix memory allocation in variant calling"
git commit -m "Update documentation for new parameters"

# ❌ Bad commit messages
git commit -m "fix"
git commit -m "update"
git commit -m "changes"
````

### Versioning with Tags
````bash
# Tag releases
git tag -a v1.0.0 -m "First stable release"
git push origin v1.0.0

# Semantic versioning: MAJOR.MINOR.PATCH
# v1.0.0 - Initial release
# v1.0.1 - Bug fix
# v1.1.0 - New feature (backward compatible)
# v2.0.0 - Breaking change
````

---

## Code Style

### Formatting
````groovy
// ✅ Good: Consistent indentation (4 spaces)
process FASTQC {
    tag "$meta.id"
    publishDir "${params.outdir}/fastqc", mode: 'copy'
    
    input:
    tuple val(meta), path(reads)
    
    output:
    path "*.html"
    
    script:
    """
    fastqc ${reads}
    """
}

// ✅ Good: Align multi-line parameters
STAR_ALIGN(
    trimmed_ch,
    star_index,
    params.gtf
)

// ✅ Good: Break long lines
def long_string = "This is a very long string that should be " +
                  "broken across multiple lines for readability"
````

### Groovy Best Practices
````groovy
// ✅ Use def for local variables
def sample_count = 10

// ✅ Use explicit types for clarity when needed
String genome = params.genome
Integer min_coverage = params.min_coverage

// ✅ Use Groovy string interpolation
println "Processing ${sample_count} samples with genome ${genome}"

// ✅ Use closures effectively
samples_ch
    .map { meta, reads -> 
        meta.new_field = "value"
        tuple(meta, reads)
    }
    .filter { meta, reads -> 
        meta.condition == 'treatment' 
    }
````

---

## Performance Best Practices

### 1. Minimize Data Movement
````groovy
// ✅ Good: Use publishDir only for final results
process INTERMEDIATE_STEP {
    // No publishDir - stays in work directory
    
    output:
    path "intermediate.txt"
}

process FINAL_STEP {
    publishDir "${params.outdir}", mode: 'copy'
    
    output:
    path "final_result.txt"
}
````

### 2. Use Efficient Channel Operators
````groovy
// ❌ Bad: Multiple view() calls slow down pipeline
channel
    .fromPath('*.txt')
    .view()
    .map { it.name }
    .view()
    .filter { it.contains('sample') }
    .view()

// ✅ Good: One view() for debugging, remove in production
channel
    .fromPath('*.txt')
    .map { it.name }
    .filter { it.contains('sample') }
    .view()  // Only if needed for debugging
````

### 3. Cache Strategy
````groovy
process EXPENSIVE_CALCULATION {
    cache 'deep'  // Hash file content, not just name
    
    input:
    path input_file
    
    output:
    path "result.txt"
    
    script:
    """
    expensive_calculation.sh ${input_file}
    """
}
````

---

## Summary Checklist

### Before Committing

- [ ] Code is properly indented and formatted
- [ ] All processes have descriptive names (UPPER_CASE)
- [ ] Containers are pinned to specific versions
- [ ] Error handling is implemented
- [ ] Resources are appropriately set
- [ ] Comments explain complex logic
- [ ] No hardcoded paths or credentials
- [ ] `.gitignore` is configured
- [ ] Tests pass

### Before Releasing

- [ ] README.md is complete
- [ ] All parameters are documented
- [ ] Example data/samplesheet provided
- [ ] CHANGELOG.md updated
- [ ] Version number updated
- [ ] Tests pass in clean environment
- [ ] CI/CD configured
- [ ] License file included

---

## 🎯 Key Takeaways

1. **Organize** - Modular structure with separate concerns
2. **Document** - Clear README, comments, and examples
3. **Validate** - Check inputs, provide helpful errors
4. **Test** - Unit tests for modules, integration tests for workflow
5. **Secure** - Never hardcode secrets, validate user input
6. **Optimize** - Efficient resource usage, minimal data movement
7. **Version** - Use Git properly with meaningful commits

---

## 📚 Further Reading

- [nf-core Best Practices](https://nf-co.re/docs/contributing/guidelines)
- [Nextflow Patterns](https://nextflow-io.github.io/patterns/)
- [Groovy Style Guide](https://groovy-lang.org/style-guide.html)

---

**Ready to debug?** → [Troubleshooting Guide](06_Troubleshooting.md)