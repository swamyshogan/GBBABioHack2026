# Nextflow Basics

Learn the fundamental concepts of Nextflow workflow management.

---

## 📖 Table of Contents

1. [What is Nextflow?](#what-is-nextflow)
2. [Core Concepts](#core-concepts)
3. [Your First Pipeline](#your-first-pipeline)
4. [Processes](#processes)
5. [Channels](#channels)
6. [Workflows](#workflows)
7. [Configuration](#configuration)
8. [Hands-On Examples](#hands-on-examples)

---

## What is Nextflow?

**Nextflow** is a workflow management system that enables:
- ✅ **Reproducibility:** Same results every time
- ✅ **Portability:** Run anywhere (laptop → HPC → cloud)
- ✅ **Scalability:** From 1 sample to 10,000 samples
- ✅ **Resume capability:** Continue from where it failed

### Why Nextflow for Bioinformatics?

**Traditional Bash Script:**
```bash
#!/bin/bash
# ❌ Hard to parallelize
# ❌ No error handling
# ❌ Environment-dependent
# ❌ Can't resume on failure

for sample in *.fastq.gz; do
    fastqc $sample
    bwa mem ref.fa $sample > $sample.sam
    samtools sort $sample.sam > $sample.bam
done
```

**Nextflow Pipeline:**
```groovy
// ✅ Automatic parallelization
// ✅ Built-in error handling
// ✅ Container-based (reproducible)
// ✅ Resume on failure with -resume

process FASTQC {
    container 'biocontainers/fastqc:v0.11.9_cv8'
    input: path(fastq)
    output: path("*_fastqc.html")
    script: "fastqc ${fastq}"
}
```

---

## Core Concepts

### 1. Processes
**Atomic units of work** - like functions in programming
```groovy
process PROCESS_NAME {
    // What goes in
    input:
    <input definition>
    
    // What comes out
    output:
    <output definition>
    
    // What to do
    script:
    """
    <your commands here>
    """
}
```

### 2. Channels
**Data flow** - how data moves between processes
```groovy
// Create a channel
Channel.fromPath('*.fastq') | VIEW

// Channels are asynchronous queues
```

### 3. Workflows
**Orchestration** - connect processes together
```groovy
workflow {
    input_ch = Channel.fromPath('*.fastq')
    FASTQC(input_ch)
    ALIGN(FASTQC.out)
}
```

---

## Your First Pipeline

Let's build a simple QC pipeline step by step.

### Step 1: Create Files
```bash
mkdir my_first_pipeline
cd my_first_pipeline
touch main.nf nextflow.config
```

### Step 2: Write `main.nf`
```groovy
#!/usr/bin/env nextflow
nextflow.enable.dsl=2

// Print a welcome message
println "Hello from Nextflow!"

// Define a process
process SAY_HELLO {
    input:
    val name
    
    output:
    stdout
    
    script:
    """
    echo "Hello, ${name}!"
    """
}

// Define the workflow
workflow {
    // Create a channel with names
    names_ch = Channel.of('Alice', 'Bob', 'Charlie')
    
    // Run the process
    SAY_HELLO(names_ch)
    
    // View the output
    SAY_HELLO.out.view()
}
```

### Step 3: Run It!
```bash
nextflow run main.nf
```

**Output:**
```
N E X T F L O W  ~  version 23.10.1
executor >  local (3)
[a1/b2c3d4] process > SAY_HELLO (1) [100%] 3 of 3 ✔
Hello, Alice!
Hello, Bob!
Hello, Charlie!
```

**What just happened?**
1. Nextflow read `main.nf`
2. Created a channel with 3 values
3. Ran the process 3 times in parallel
4. Printed the outputs

---

## Processes

### Anatomy of a Process
```groovy
process PROCESS_NAME {
    // Directives (optional)
    tag "$sample_id"
    publishDir "results/", mode: 'copy'
    container 'biocontainers/tool:version'
    cpus 4
    memory '8 GB'
    
    // Input (what data comes in)
    input:
    tuple val(sample_id), path(reads)
    path reference
    
    // Output (what data goes out)
    output:
    tuple val(sample_id), path("${sample_id}.bam")
    
    // Script (what to execute)
    script:
    """
    tool --input ${reads} \\
         --reference ${reference} \\
         --output ${sample_id}.bam \\
         --threads ${task.cpus}
    """
}
```

### Common Input Types
```groovy
input:
val(x)              // A value (string, number, etc.)
path(file)          // A file or directory
tuple val(id), path(file)  // Multiple values together
each item           // Repeat process for each item
```

### Common Output Types
```groovy
output:
path("*.txt")                    // File(s) matching pattern
tuple val(id), path("*.bam")     // Multiple outputs
stdout                           // Standard output
path("results/*"), emit: results // Named output
```

### Example: FastQC Process
```groovy
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
```

---

## Channels

Channels are **asynchronous queues** that connect processes.

### Creating Channels
```groovy
// From a list
Channel.of(1, 2, 3, 4, 5)

// From files
Channel.fromPath('data/*.fastq.gz')

// From file pairs (paired-end reads)
Channel.fromFilePairs('data/*_R{1,2}.fastq.gz')

// From CSV
Channel.fromPath('samplesheet.csv')
    .splitCsv(header: true)
    .map { row -> tuple(row.sample, row.fastq) }
```

### Channel Operators
```groovy
// View channel content (for debugging)
channel.view()

// Map - transform each item
channel.map { it * 2 }

// Filter - select items
channel.filter { it > 10 }

// Collect - gather all items into a list
channel.collect()

// GroupTuple - group by key
channel.groupTuple()

// Join - combine two channels by matching key
channel1.join(channel2)

// Mix - merge multiple channels
channel1.mix(channel2)
```

### Example: Processing Paired-End Reads
```groovy
// Input: sample1_R1.fastq.gz, sample1_R2.fastq.gz

Channel
    .fromFilePairs('data/*_R{1,2}.fastq.gz')
    .view()

// Output:
// [sample1, [/path/sample1_R1.fastq.gz, /path/sample1_R2.fastq.gz]]
```

---

## Workflows

Workflows connect processes together.

### Basic Workflow
```groovy
workflow {
    // Create input channel
    reads_ch = Channel.fromPath('*.fastq.gz')
    
    // Run processes in order
    FASTQC(reads_ch)
    TRIM(reads_ch)
    ALIGN(TRIM.out)
}
```

### Named Workflows (Modular)
```groovy
workflow QC_WORKFLOW {
    take:
    reads
    
    main:
    FASTQC(reads)
    MULTIQC(FASTQC.out.collect())
    
    emit:
    qc_report = MULTIQC.out
}

workflow {
    reads_ch = Channel.fromPath('*.fastq.gz')
    QC_WORKFLOW(reads_ch)
}
```

---

## Configuration

Configuration separates **logic** (what to do) from **execution** (how to do it).

### nextflow.config
```groovy
// Parameters
params {
    input = 'data/*.fastq.gz'
    outdir = 'results'
    genome = 'hg38'
}

// Process defaults
process {
    cpus = 2
    memory = '4 GB'
    time = '1 h'
    container = 'ubuntu:20.04'
}

// Specific process settings
process {
    withName: ALIGN {
        cpus = 8
        memory = '32 GB'
    }
}

// Profiles
profiles {
    standard {
        process.executor = 'local'
    }
    
    docker {
        docker.enabled = true
        docker.runOptions = '-u $(id -u):$(id -g)'
    }
    
    test {
        params.input = 'test_data/*.fastq.gz'
    }
}

// Reports
timeline {
    enabled = true
    file = "${params.outdir}/timeline.html"
}

report {
    enabled = true
    file = "${params.outdir}/report.html"
}
```

### Using Profiles
```bash
# Use docker profile
nextflow run main.nf -profile docker

# Use multiple profiles
nextflow run main.nf -profile test,docker

# Override parameters
nextflow run main.nf --input 'my_data/*.fastq'
```

---

## Hands-On Examples

### Example 1: Simple QC Pipeline
```groovy
#!/usr/bin/env nextflow
nextflow.enable.dsl=2

params.reads = 'data/*_R{1,2}.fastq.gz'
params.outdir = 'results'

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

process MULTIQC {
    publishDir "${params.outdir}/multiqc", mode: 'copy'
    container 'ewels/multiqc:v1.14'
    
    input:
    path '*'
    
    output:
    path "multiqc_report.html"
    
    script:
    """
    multiqc .
    """
}

workflow {
    reads_ch = Channel
        .fromFilePairs(params.reads, checkIfExists: true)
    
    FASTQC(reads_ch)
    MULTIQC(FASTQC.out.collect())
}
```

### Example 2: Using CSV Input

**samplesheet.csv:**
```csv
sample,fastq_1,fastq_2,condition
sample1,data/sample1_R1.fastq.gz,data/sample1_R2.fastq.gz,control
sample2,data/sample2_R1.fastq.gz,data/sample2_R2.fastq.gz,treatment
```

**Pipeline:**
```groovy
params.input = 'samplesheet.csv'

workflow {
    Channel
        .fromPath(params.input)
        .splitCsv(header: true)
        .map { row -> 
            tuple(
                row.sample, 
                [file(row.fastq_1), file(row.fastq_2)],
                row.condition
            )
        }
        .set { samples_ch }
    
    FASTQC(samples_ch)
}
```

---

## 🎯 Key Takeaways

1. **Processes** = What to do (atomic units)
2. **Channels** = How data flows (asynchronous queues)
3. **Workflows** = Orchestration (connecting processes)
4. **Configuration** = Execution settings (separate from logic)

---

## 🧪 Practice Exercises

### Exercise 1: Modify SAY_HELLO
Add your own names to the channel and run it.

### Exercise 2: Create a GOODBYE Process
Create a new process that says goodbye instead of hello.

### Exercise 3: Chain Processes
Make GOODBYE run after SAY_HELLO using the same names.

---

## 📚 Next Steps

- ✅ [Advanced Nextflow](03_Nextflow_Advanced.md)
- ✅ [Containers Guide](04_Containers.md)
- ✅ [Best Practices](05_Best_Practices.md)

---

## 📖 Additional Resources

- [Official Nextflow Documentation](https://nextflow.io/docs/latest/)
- [Nextflow Training](https://training.nextflow.io/)
- [nf-core Pipelines](https://nf-co.re/) - Examples of production pipelines

---

**Ready for more?** → [Advanced Nextflow](03_Nextflow_Advanced.md)