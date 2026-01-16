# Advanced Nextflow Techniques

Take your Nextflow skills to the next level with advanced patterns and optimization strategies.

---

## 📖 Table of Contents

1. [Channel Operators Deep Dive](#channel-operators-deep-dive)
2. [Dynamic Resource Allocation](#dynamic-resource-allocation)
3. [Error Handling & Retry Strategies](#error-handling--retry-strategies)
4. [Modular Pipeline Design](#modular-pipeline-design)
5. [Conditional Execution](#conditional-execution)
6. [Performance Optimization](#performance-optimization)
7. [Advanced Configuration](#advanced-configuration)

---

## Channel Operators Deep Dive

### map() - Transform Data
```groovy
// Basic transformation
Channel.of(1, 2, 3, 4, 5)
    .map { it * 2 }
    .view()
// Output: 2, 4, 6, 8, 10

// Extract specific fields from CSV
Channel
    .fromPath('samplesheet.csv')
    .splitCsv(header: true)
    .map { row -> tuple(row.sample, file(row.fastq_1), file(row.fastq_2)) }
    .view()

// Add metadata
Channel
    .fromFilePairs('data/*_R{1,2}.fastq.gz')
    .map { sample_id, files -> 
        tuple(sample_id, files, "batch_1") 
    }
    .view()
```

### filter() - Select Specific Items
```groovy
// Filter by condition
Channel.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
    .filter { it % 2 == 0 }  // Keep only even numbers
    .view()
// Output: 2, 4, 6, 8, 10

// Filter samples by condition
Channel
    .fromPath('samplesheet.csv')
    .splitCsv(header: true)
    .filter { row -> row.condition == 'treatment' }
    .map { row -> tuple(row.sample, file(row.fastq)) }
    .view()
```

### groupTuple() - Group by Key
```groovy
// Group technical replicates
Channel.of(
    ['sample1', 'file1_rep1.bam'],
    ['sample1', 'file1_rep2.bam'],
    ['sample2', 'file2_rep1.bam'],
    ['sample2', 'file2_rep2.bam']
)
.groupTuple()
.view()
// Output:
// [sample1, [file1_rep1.bam, file1_rep2.bam]]
// [sample2, [file2_rep1.bam, file2_rep2.bam]]

// Group by size
Channel.of(
    ['sample1', 'file1.txt'],
    ['sample2', 'file2.txt'],
    ['sample1', 'file3.txt']
)
.groupTuple(size: 2)  // Wait for exactly 2 items per group
.view()
```

### join() - Combine Channels by Key
```groovy
// Join RNA-seq and methylation data
rnaseq_ch = Channel.of(
    ['sample1', 'rnaseq1.bam'],
    ['sample2', 'rnaseq2.bam']
)

methyl_ch = Channel.of(
    ['sample1', 'methyl1.bed'],
    ['sample2', 'methyl2.bed']
)

rnaseq_ch
    .join(methyl_ch)
    .view()
// Output:
// [sample1, rnaseq1.bam, methyl1.bed]
// [sample2, rnaseq2.bam, methyl2.bed]
```

### branch() - Split Channel into Multiple Channels
```groovy
Channel
    .fromPath('samplesheet.csv')
    .splitCsv(header: true)
    .map { row -> tuple(row.sample, file(row.fastq), row.condition) }
    .branch {
        control: it[2] == 'control'
        treatment: it[2] == 'treatment'
    }
    .set { branched_ch }

// Use each branch separately
branched_ch.control.view { "Control: $it" }
branched_ch.treatment.view { "Treatment: $it" }
```

### combine() - Cartesian Product
```groovy
// Test multiple parameter combinations
samples_ch = Channel.of('sample1', 'sample2')
params_ch = Channel.of('param_a', 'param_b')

samples_ch
    .combine(params_ch)
    .view()
// Output:
// [sample1, param_a]
// [sample1, param_b]
// [sample2, param_a]
// [sample2, param_b]
```

### collect() - Gather All Items
```groovy
// Collect all QC reports for MultiQC
Channel
    .fromPath('results/fastqc/*.zip')
    .collect()
    .view()
// Output: [file1.zip, file2.zip, file3.zip, ...]

// Use in workflow
workflow {
    FASTQC(reads_ch)
    MULTIQC(FASTQC.out.collect())  // Wait for all FastQC to finish
}
```

### flatten() - Unwrap Lists
```groovy
Channel.of([1, 2], [3, 4], [5])
    .flatten()
    .view()
// Output: 1, 2, 3, 4, 5
```

### Complex Example: Multi-Step Data Processing
```groovy
Channel
    .fromPath('samplesheet.csv')
    .splitCsv(header: true)
    .map { row -> 
        tuple(
            row.sample,
            file(row.fastq_1),
            file(row.fastq_2),
            row.condition,
            row.batch
        )
    }
    .filter { sample, r1, r2, condition, batch -> 
        batch == 'batch_1'  // Only process batch 1
    }
    .map { sample, r1, r2, condition, batch -> 
        tuple(sample, [r1, r2], condition)  // Simplify tuple
    }
    .set { filtered_ch }
```

---

## Dynamic Resource Allocation

### Basic Dynamic Allocation
```groovy
process ALIGN {
    cpus { 4 * task.attempt }
    memory { 8.GB * task.attempt }
    time { 2.h * task.attempt }
    
    errorStrategy 'retry'
    maxRetries 3
    
    input:
    tuple val(sample), path(reads)
    
    script:
    """
    echo "Attempt ${task.attempt} using ${task.cpus} CPUs and ${task.memory}"
    bwa mem -t ${task.cpus} reference.fa ${reads} > ${sample}.sam
    """
}
```

### File Size-Based Resources
```groovy
process SORT_BAM {
    cpus 4
    memory { 4.GB * Math.ceil(bam.size() / 1000000000) }  // 4GB per GB of input
    
    input:
    path bam
    
    script:
    """
    samtools sort -@ ${task.cpus} -m ${task.memory.toGiga()/task.cpus}G ${bam}
    """
}
```

### Conditional Resources
```groovy
process VARIANT_CALL {
    cpus { params.fastmode ? 4 : 16 }
    memory { params.fastmode ? 8.GB : 64.GB }
    
    input:
    tuple val(sample), path(bam)
    
    script:
    """
    gatk HaplotypeCaller \\
        -I ${bam} \\
        -O ${sample}.vcf \\
        --native-pair-hmm-threads ${task.cpus}
    """
}
```

---

## Error Handling & Retry Strategies

### Basic Error Strategy
```groovy
process FLAKY_PROCESS {
    errorStrategy 'retry'
    maxRetries 3
    
    script:
    """
    # Attempt might fail, will retry up to 3 times
    """
}
```

### Different Strategies by Exit Code
```groovy
process SMART_RETRY {
    errorStrategy { task.exitStatus in 137..140 ? 'retry' : 'terminate' }
    maxRetries 3
    
    // 137-140 are memory-related errors, retry with more memory
    memory { 8.GB * task.attempt }
    
    script:
    """
    # Your command here
    """
}
```

### Ignore Certain Failures
```groovy
process OPTIONAL_QC {
    errorStrategy 'ignore'  // Continue pipeline even if this fails
    
    script:
    """
    # Optional quality check
    """
}
```

### Custom Error Handling
```groovy
process ROBUST_PROCESS {
    errorStrategy { task.attempt <= 2 ? 'retry' : 'ignore' }
    maxRetries 2
    
    input:
    tuple val(sample), path(file)
    
    script:
    """
    # Try processing
    process_file.sh ${file} || {
        echo "Processing failed for ${sample}"
        # Create empty output so pipeline continues
        touch ${sample}.failed.txt
    }
    """
}
```

### Afterscript for Cleanup
```groovy
process WITH_CLEANUP {
    errorStrategy 'retry'
    maxRetries 3
    
    script:
    """
    # Main processing
    big_analysis.sh input.txt
    """
    
    stub:
    """
    # For testing
    touch output.txt
    """
}
```

---

## Modular Pipeline Design

### Organizing with Modules

**Directory structure:**
```
pipeline/
├── main.nf
├── nextflow.config
├── modules/
│   ├── fastqc.nf
│   ├── trimming.nf
│   ├── alignment.nf
│   └── quantification.nf
└── subworkflows/
    └── qc_workflow.nf
```

### Creating a Module

**modules/fastqc.nf:**
```groovy
process FASTQC {
    tag "$sample_id"
    publishDir "${params.outdir}/fastqc", mode: 'copy'
    container 'biocontainers/fastqc:v0.11.9_cv8'
    
    input:
    tuple val(sample_id), path(reads)
    
    output:
    path "*.{html,zip}", emit: reports
    
    script:
    """
    fastqc -q -t ${task.cpus} ${reads}
    """
}
```

### Including Modules in Main Pipeline

**main.nf:**
```groovy
#!/usr/bin/env nextflow
nextflow.enable.dsl=2

// Import modules
include { FASTQC } from './modules/fastqc'
include { TRIMMOMATIC } from './modules/trimming'
include { STAR_ALIGN } from './modules/alignment'

workflow {
    reads_ch = Channel.fromFilePairs(params.reads)
    
    FASTQC(reads_ch)
    TRIMMOMATIC(reads_ch)
    STAR_ALIGN(TRIMMOMATIC.out)
}
```

### Creating Subworkflows

**subworkflows/qc_workflow.nf:**
```groovy
include { FASTQC } from '../modules/fastqc'
include { MULTIQC } from '../modules/multiqc'

workflow QC_WORKFLOW {
    take:
    reads  // Input channel
    
    main:
    FASTQC(reads)
    MULTIQC(FASTQC.out.reports.collect())
    
    emit:
    qc_report = MULTIQC.out
    fastqc_reports = FASTQC.out.reports
}
```

**Using subworkflows:**
```groovy
include { QC_WORKFLOW } from './subworkflows/qc_workflow'

workflow {
    reads_ch = Channel.fromFilePairs(params.reads)
    QC_WORKFLOW(reads_ch)
}
```

---

## Conditional Execution

### Using when Directive
```groovy
process OPTIONAL_STEP {
    when:
    params.run_optional
    
    input:
    path input_file
    
    script:
    """
    optional_analysis.sh ${input_file}
    """
}
```

### Conditional in Workflow
```groovy
workflow {
    reads_ch = Channel.fromFilePairs(params.reads)
    
    FASTQC(reads_ch)
    
    if (params.trim) {
        TRIMMOMATIC(reads_ch)
        aligned_ch = TRIMMOMATIC.out
    } else {
        aligned_ch = reads_ch
    }
    
    ALIGN(aligned_ch)
}
```

### Branch-Based Conditional Processing
```groovy
workflow {
    samples_ch = Channel
        .fromPath(params.input)
        .splitCsv(header: true)
        .map { row -> tuple(row.sample, file(row.fastq), row.type) }
        .branch {
            rna: it[2] == 'rna'
            dna: it[2] == 'dna'
        }
    
    // Process RNA and DNA differently
    RNA_WORKFLOW(samples_ch.rna)
    DNA_WORKFLOW(samples_ch.dna)
}
```

---

## Performance Optimization

### Parallel Processing with Scatter-Gather
```groovy
// Split work into chunks
process SPLIT_INTERVALS {
    input:
    path genome
    
    output:
    path "interval_*.bed"
    
    script:
    """
    split_genome.sh ${genome} ${params.n_chunks}
    """
}

// Process each chunk in parallel
process VARIANT_CALL_CHUNK {
    input:
    tuple val(sample), path(bam), path(interval)
    
    output:
    tuple val(sample), path("${sample}.${interval}.vcf")
    
    script:
    """
    gatk HaplotypeCaller \\
        -I ${bam} \\
        -L ${interval} \\
        -O ${sample}.${interval}.vcf
    """
}

// Merge results
process MERGE_VCFS {
    input:
    tuple val(sample), path(vcfs)
    
    output:
    path "${sample}.merged.vcf"
    
    script:
    """
    bcftools concat ${vcfs} -o ${sample}.merged.vcf
    """
}

workflow {
    bam_ch = Channel.fromPath('*.bam').map { [it.baseName, it] }
    
    SPLIT_INTERVALS(params.genome)
    
    // Combine each BAM with each interval
    bam_ch
        .combine(SPLIT_INTERVALS.out.flatten())
        .set { chunks_ch }
    
    VARIANT_CALL_CHUNK(chunks_ch)
    
    // Group VCFs by sample
    VARIANT_CALL_CHUNK.out
        .groupTuple()
        .set { grouped_vcfs }
    
    MERGE_VCFS(grouped_vcfs)
}
```

### Optimizing I/O
```groovy
process INTENSIVE_PROCESS {
    scratch true  // Use local scratch space
    stageInMode 'copy'  // or 'symlink'
    stageOutMode 'move'
    
    input:
    path large_file
    
    script:
    """
    # Processing happens in local scratch
    # Faster I/O, then moves results back
    """
}
```

### Caching and Reusing Results
```groovy
process EXPENSIVE_CALCULATION {
    cache 'deep'  // Cache based on file content, not just name
    
    input:
    path input_file
    
    output:
    path "result.txt"
    
    script:
    """
    expensive_calculation.sh ${input_file} > result.txt
    """
}
```

### Resource Labels for Grouping
```groovy
// In nextflow.config
process {
    withLabel: 'low_mem' {
        cpus = 2
        memory = 4.GB
    }
    
    withLabel: 'high_mem' {
        cpus = 8
        memory = 64.GB
    }
}

// In process definition
process ALIGNMENT {
    label 'high_mem'
    
    script:
    """
    bwa mem ...
    """
}
```

---

## Advanced Configuration

### Environment-Specific Profiles

**nextflow.config:**
```groovy
profiles {
    standard {
        process.executor = 'local'
        process.cpus = 4
        process.memory = '8 GB'
    }
    
    laptop {
        process.executor = 'local'
        process.cpus = 2
        process.memory = '4 GB'
        docker.enabled = true
    }
    
    workstation {
        process.executor = 'local'
        process.cpus = 16
        process.memory = '64 GB'
        docker.enabled = true
    }
    
    cluster {
        process.executor = 'slurm'
        process.queue = 'general'
        process.cpus = 8
        process.memory = '32 GB'
        singularity.enabled = true
    }
    
    aws {
        process.executor = 'awsbatch'
        process.queue = 'my-batch-queue'
        aws.region = 'us-east-1'
        docker.enabled = true
    }
}
```

### Custom Configuration Files

**conf/resources.config:**
```groovy
process {
    withName: FASTQC {
        cpus = 2
        memory = 4.GB
    }
    
    withName: STAR_ALIGN {
        cpus = 16
        memory = 64.GB
        time = 4.h
    }
    
    withName: 'DESEQ2.*' {  // Pattern matching
        cpus = 4
        memory = 16.GB
    }
}
```

**Include in main config:**
```groovy
// nextflow.config
includeConfig 'conf/resources.config'
```

### Parameter Validation
```groovy
// At top of main.nf
def checkParams() {
    if (!params.input) {
        error "Please provide --input parameter"
    }
    
    if (!file(params.input).exists()) {
        error "Input file does not exist: ${params.input}"
    }
    
    if (params.genome !in ['hg38', 'hg19', 'mm10']) {
        error "Invalid genome: ${params.genome}. Must be hg38, hg19, or mm10"
    }
}

checkParams()

workflow {
    // Your workflow
}
```

### Parameter Schema (JSON)

**nextflow_schema.json:**
```json
{
    "title": "My Pipeline Parameters",
    "type": "object",
    "properties": {
        "input": {
            "type": "string",
            "description": "Path to input samplesheet",
            "format": "file-path"
        },
        "outdir": {
            "type": "string",
            "default": "results",
            "description": "Output directory"
        },
        "genome": {
            "type": "string",
            "enum": ["hg38", "hg19", "mm10"],
            "description": "Reference genome"
        }
    },
    "required": ["input"]
}
```

---

## Advanced Patterns

### Pattern 1: Fan-Out, Fan-In
```groovy
// One input creates multiple outputs, then merge
workflow {
    input_ch = Channel.of('sample1')
    
    // Fan-out: Create multiple analyses
    ANALYSIS_A(input_ch)
    ANALYSIS_B(input_ch)
    ANALYSIS_C(input_ch)
    
    // Fan-in: Merge results
    ANALYSIS_A.out
        .mix(ANALYSIS_B.out, ANALYSIS_C.out)
        .collect()
        .set { merged_ch }
    
    FINAL_REPORT(merged_ch)
}
```

### Pattern 2: Checkpoint-Resume
```groovy
process CHECKPOINT {
    storeDir "${params.outdir}/checkpoints"  // Persist results
    
    input:
    path input
    
    output:
    path "checkpoint.txt"
    
    script:
    """
    # Long-running process
    # Results stored permanently, not in work/ directory
    """
}
```

### Pattern 3: Metadata Tracking
```groovy
workflow {
    Channel
        .fromPath(params.input)
        .splitCsv(header: true)
        .map { row ->
            def meta = [
                id: row.sample,
                condition: row.condition,
                batch: row.batch,
                paired: true
            ]
            tuple(meta, [file(row.fastq_1), file(row.fastq_2)])
        }
        .set { samples_ch }
    
    // Metadata flows through pipeline
    FASTQC(samples_ch)
    ALIGN(samples_ch)
}
```

---

## 🎯 Best Practices Summary

1. **Use channel operators effectively** - Transform data in channels, not in processes
2. **Implement retry strategies** - Make pipelines robust to transient failures
3. **Modularize your code** - Separate processes, workflows, and configs
4. **Optimize resource usage** - Dynamic allocation based on input size
5. **Use profiles** - One pipeline, multiple execution environments
6. **Cache expensive steps** - Save time on re-runs
7. **Validate inputs** - Fail fast with clear error messages

---

## 🧪 Practice Exercises

### Exercise 1: Dynamic Resources
Create a process that allocates memory based on input file size.

### Exercise 2: Scatter-Gather
Implement a pipeline that splits work into 4 chunks, processes in parallel, then merges.

### Exercise 3: Conditional Workflow
Build a workflow that branches based on sample type (RNA vs DNA).

---

## 📚 Next Steps

- ✅ [Containers Guide](04_Containers.md)
- ✅ [Best Practices](05_Best_Practices.md)
- ✅ [Troubleshooting](06_Troubleshooting.md)

---

**Ready for containers?** → [Containers Guide](04_Containers.md)