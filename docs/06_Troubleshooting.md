# Troubleshooting Guide

Solutions to common Nextflow problems and debugging strategies.

---

## 📖 Table of Contents

1. [General Debugging Strategy](#general-debugging-strategy)
2. [Installation Issues](#installation-issues)
3. [Pipeline Execution Errors](#pipeline-execution-errors)
4. [Container Issues](#container-issues)
5. [Resource Problems](#resource-problems)
6. [Channel & Data Flow Issues](#channel--data-flow-issues)
7. [Performance Issues](#performance-issues)
8. [Getting Help](#getting-help)

---

## General Debugging Strategy

### Step 1: Read the Error Message

Nextflow provides detailed error messages. **Read them carefully!**
````
Error executing process > 'FASTQC (sample1)'

Caused by:
  Process `FASTQC (sample1)` terminated with an error exit status (1)

Command executed:
  fastqc -q sample1_R1.fastq.gz

Command exit status:
  1

Command output:
  (empty)

Command error:
  fastqc: command not found

Work dir:
  /path/to/work/a1/b2c3d4e5f6g7h8i9
````

**Key information:**
- ✅ Which process failed: `FASTQC (sample1)`
- ✅ Exit status: `1` (non-zero = error)
- ✅ Error message: `fastqc: command not found`
- ✅ Work directory: Where to investigate further

### Step 2: Navigate to Work Directory
````bash
# Go to the work directory shown in error
cd /path/to/work/a1/b2c3d4e5f6g7h8i9

# List contents
ls -la

# Key files:
# .command.sh    - The script that was executed
# .command.log   - Standard output
# .command.err   - Standard error
# .command.run   - Wrapper script
# .exitcode      - Exit code of the process
````

### Step 3: Examine the Files
````bash
# See what command was run
cat .command.sh

# Check standard output
cat .command.log

# Check error output
cat .command.err

# Check exit code
cat .exitcode
````

### Step 4: Try Running Manually
````bash
# Try executing the command yourself
bash .command.sh

# This often reveals the actual problem
````

### Step 5: Fix and Resume
````bash
# Fix the issue in your pipeline code
# Then resume from where it failed

nextflow run main.nf -resume
````

---

## Installation Issues

### Issue: "nextflow: command not found"

**Symptoms:**
````bash
$ nextflow run hello
bash: nextflow: command not found
````

**Solutions:**

**Option 1: Add to PATH**
````bash
# Find where nextflow is
which nextflow

# If not found, download it
curl -s https://get.nextflow.io | bash

# Move to a directory in PATH
sudo mv nextflow /usr/local/bin/

# Or add current directory to PATH
export PATH=$PATH:$PWD
echo 'export PATH=$PATH:$PWD' >> ~/.bashrc
````

**Option 2: Use full path**
````bash
./nextflow run hello
````

**Option 3: Create symlink**
````bash
ln -s /path/to/nextflow /usr/local/bin/nextflow
````

---

### Issue: "java.lang.UnsupportedClassVersionError"

**Symptoms:**
````
Error: A JNI error has occurred
java.lang.UnsupportedClassVersionError: nextflow/cli/Launcher has been compiled by a more recent version of the Java Runtime
````

**Cause:** Nextflow requires Java 11 or later

**Solution:**
````bash
# Check Java version
java -version

# Should show version 11 or higher
# If not, install newer Java

# Ubuntu/Debian
sudo apt-get install openjdk-11-jdk

# macOS
brew install openjdk@11

# Verify
java -version
````

---

### Issue: "Unable to initialize nextflow environment"

**Symptoms:**
````
Unable to initialize nextflow environment
````

**Solutions:**

1. **Check internet connection** (Nextflow needs to download dependencies)

2. **Clear Nextflow cache:**
````bash
rm -rf ~/.nextflow
nextflow info
````

3. **Check disk space:**
````bash
df -h
````

4. **Update Nextflow:**
````bash
nextflow self-update
````

---

## Pipeline Execution Errors

### Issue: "Process terminated with an error exit status"

**Symptoms:**
````
Process `FASTQC` terminated with an error exit status (127)
````

**Common exit codes:**
- `1` - General error
- `127` - Command not found
- `130` - Process terminated by user (Ctrl+C)
- `137` - Out of memory (killed by system)
- `139` - Segmentation fault
- `143` - Terminated by timeout

**Debugging steps:**
````bash
# Navigate to work directory
cd work/a1/b2c3d4e5f6/

# Check what happened
cat .command.err
cat .command.log

# Try running the command manually
bash .command.sh
````

---

### Issue: "Command not found" (exit 127)

**Symptoms:**
````
Command error:
  fastqc: command not found
````

**Causes & Solutions:**

**1. Tool not in container:**
````groovy
// ❌ Missing container
process FASTQC {
    script:
    """
    fastqc input.fastq
    """
}

// ✅ Add container
process FASTQC {
    container 'biocontainers/fastqc:v0.11.9_cv8'
    
    script:
    """
    fastqc input.fastq
    """
}
````

**2. Container not enabled:**
````bash
# Enable Docker
nextflow run main.nf -with-docker

# Or in config
docker.enabled = true
````

**3. Wrong container:**
````groovy
// Check container actually has the tool
docker run biocontainers/fastqc:v0.11.9_cv8 which fastqc
````

---

### Issue: Input file not found

**Symptoms:**
````
No such file or directory: /path/to/input.fastq
````

**Solutions:**

**1. Check file path:**
````groovy
// Use checkIfExists
Channel
    .fromPath(params.input, checkIfExists: true)
    .set { input_ch }
````

**2. Check wildcards:**
````groovy
// ❌ Wrong pattern
Channel.fromPath('data/*.fq')  // Looking for .fq

// ✅ Correct pattern
Channel.fromPath('data/*.fastq.gz')  // Files are .fastq.gz
````

**3. Use absolute paths in samplesheet:**
````csv
sample,fastq
sample1,/full/path/to/sample1.fastq.gz
````

**4. Check current directory:**
````bash
# Nextflow runs from where you execute it
pwd

# Files are relative to this location
ls data/*.fastq.gz
````

---

### Issue: "No such variable" error

**Symptoms:**
````
No such variable: reads
````

**Causes & Solutions:**

**1. Typo in variable name:**
````groovy
// ❌ Typo
input:
path read

script:
"""
fastqc ${reads}  // reads vs read
"""

// ✅ Fixed
input:
path reads

script:
"""
fastqc ${reads}
"""
````

**2. Wrong scope:**
````groovy
// ❌ Variable not in scope
process A {
    output:
    path result
    
    script:
    def my_var = "test"
    """
    echo ${my_var}
    """
}

process B {
    script:
    """
    echo ${my_var}  // my_var not defined here!
    """
}
````

---

## Container Issues

### Issue: "Cannot connect to Docker daemon"

**Symptoms:**
````
Cannot connect to the Docker daemon at unix:///var/run/docker.sock
````

**Solutions:**

**1. Start Docker:**
````bash
# macOS/Windows: Start Docker Desktop

# Linux: Start Docker service
sudo systemctl start docker
sudo systemctl status docker
````

**2. Add user to docker group:**
````bash
sudo usermod -aG docker $USER

# Log out and back in
# Or run:
newgrp docker
````

**3. Check Docker is running:**
````bash
docker ps
````

---

### Issue: "Permission denied" with containers

**Symptoms:**
````
Permission denied: cannot write to output directory
````

**Solution:** Run container as current user
````groovy
// nextflow.config
docker {
    enabled = true
    runOptions = '-u $(id -u):$(id -g)'
}
````

---

### Issue: Container pulls too slow

**Symptoms:**
Container download takes forever

**Solutions:**

**1. Pre-pull containers:**
````bash
# Pull all containers before running pipeline
docker pull biocontainers/fastqc:v0.11.9_cv8
docker pull biocontainers/star:2.7.10b
````

**2. Use local cache:**
````groovy
// nextflow.config
singularity {
    cacheDir = '/path/to/fast/storage/cache'
}
````

**3. Use closer registry:**
````groovy
// Use quay.io instead of Docker Hub
container 'quay.io/biocontainers/fastqc:0.11.9--0'
````

---

### Issue: "Unable to pull image"

**Symptoms:**
````
Unable to pull image `biocontainers/fastqc:v0.11.9_cv8`
````

**Solutions:**

**1. Check image name:**
````bash
# Search for correct image
docker search fastqc

# Check on BioContainers website
# https://biocontainers.pro/
````

**2. Check network:**
````bash
# Test pulling manually
docker pull biocontainers/fastqc:v0.11.9_cv8
````

**3. Use alternative registry:**
````groovy
// Try quay.io
container 'quay.io/biocontainers/fastqc:0.11.9--0'
````

---

## Resource Problems

### Issue: "Out of memory" (exit 137)

**Symptoms:**
````
Process terminated with an error exit status (137)
Command error:
  Killed
````

**Exit code 137** = killed by system (usually out of memory)

**Solutions:**

**1. Increase memory:**
````groovy
process MEMORY_HUNGRY {
    memory '32 GB'  // Increase from default
    
    script:
    """
    big_analysis.sh
    """
}
````

**2. Implement retry with more memory:**
````groovy
process SMART_RETRY {
    memory { 8.GB * task.attempt }
    errorStrategy 'retry'
    maxRetries 3
    
    script:
    """
    big_analysis.sh
    """
}
````

**3. Check system resources:**
````bash
# Check available memory
free -h

# Check what's using memory
top
htop
````

**4. Process smaller chunks:**
````groovy
// Split large file into smaller chunks
// Process each chunk separately
````

---

### Issue: Pipeline runs slow

**Symptoms:**
Pipeline takes much longer than expected

**Debugging:**

**1. Check timeline report:**
````bash
nextflow run main.nf

# Open timeline.html in browser
open timeline.html
````

Look for:
- Long-running processes
- Processes waiting for resources
- Sequential vs parallel execution

**2. Check resource utilization:**
````bash
# During pipeline execution
htop

# Check CPU usage
mpstat 1

# Check I/O
iostat 1
````

**3. Common bottlenecks:**
````groovy
// ❌ Bad: Sequential processing
workflow {
    samples.each { sample ->
        PROCESS_A(sample)
        PROCESS_B(PROCESS_A.out)
    }
}

// ✅ Good: Parallel processing
workflow {
    PROCESS_A(samples)
    PROCESS_B(PROCESS_A.out)
}
````

**4. Enable parallelization:**
````groovy
// nextflow.config
executor {
    queueSize = 10  // Submit up to 10 jobs at once
}

process {
    maxForks = 4  // Run max 4 instances per process
}
````

---

### Issue: "No space left on device"

**Symptoms:**
````
java.io.IOException: No space left on device
````

**Solutions:**

**1. Check disk space:**
````bash
df -h
````

**2. Clean up work directory:**
````bash
# Remove old pipeline runs
rm -rf work/

# Or use Nextflow's clean
nextflow clean -f
nextflow clean -f -k  # Keep latest run
````

**3. Change work directory:**
````bash
# Use disk with more space
nextflow run main.nf -w /path/to/large/disk/work
````

**4. Use scratch space:**
````groovy
process BIG_FILES {
    scratch true  // Use local scratch, not shared storage
    
    script:
    """
    # Processing happens in local scratch
    """
}
````

---

## Channel & Data Flow Issues

### Issue: "Channel is empty"

**Symptoms:**
````
Channel `input_ch` is empty
````

**Debugging:**
````groovy
// Add .view() to see channel contents
Channel
    .fromPath(params.input)
    .view { "Found file: $it" }
    .set { input_ch }

// Use .ifEmpty to catch empty channels
Channel
    .fromPath(params.input)
    .ifEmpty { error "No input files found!" }
    .set { input_ch }
````

**Common causes:**

**1. Wrong file pattern:**
````groovy
// ❌ Files are *.fastq.gz, but looking for *.fq
Channel.fromPath('data/*.fq')

// ✅ Correct pattern
Channel.fromPath('data/*.fastq.gz')
````

**2. Wrong directory:**
````bash
# Check where you're running from
pwd

# Check if files exist
ls data/*.fastq.gz
````

---

### Issue: "Duplicate process invocation"

**Symptoms:**
````
Process already defined: FASTQC
````

**Cause:** Including same module multiple times

**Solution:**
````groovy
// ❌ Bad: Including multiple times
include { FASTQC } from './modules/fastqc'
include { FASTQC } from './modules/fastqc'  // Duplicate!

// ✅ Good: Include once, use multiple times
include { FASTQC } from './modules/fastqc'

workflow {
    FASTQC(reads_ch_1)
    FASTQC(reads_ch_2)  // OK to call multiple times
}

// ✅ Or use alias
include { FASTQC as FASTQC_RAW } from './modules/fastqc'
include { FASTQC as FASTQC_TRIMMED } from './modules/fastqc'
````

---

### Issue: Data not flowing between processes

**Symptoms:**
Process B doesn't receive output from Process A

**Debugging:**
````groovy
// Check outputs with .view()
process PROCESS_A {
    output:
    path "output.txt", emit: result
    
    script:
    """
    echo "test" > output.txt
    """
}

workflow {
    PROCESS_A()
    PROCESS_A.out.result.view { "Process A output: $it" }
    PROCESS_B(PROCESS_A.out.result)
}
````

**Common issues:**

**1. Wrong output emission:**
````groovy
// ❌ Not emitting properly
process A {
    output:
    path "result.txt"
}

workflow {
    A()
    B(A.out.result)  // .result doesn't exist!
}

// ✅ Use emit
process A {
    output:
    path "result.txt", emit: result
}

workflow {
    A()
    B(A.out.result)  // Now works!
}
````

**2. Type mismatch:**
````groovy
// Process A outputs: path
// Process B expects: tuple val, path

// Need to transform:
PROCESS_A()
PROCESS_A.out
    .map { file -> tuple("sample", file) }
    .set { processed_ch }
PROCESS_B(processed_ch)
````

---

## Configuration Issues

### Issue: Parameters not being read

**Symptoms:**
````
Parameter --input is null
````

**Debugging:**
````bash
# Check parameter value
nextflow run main.nf --input test.csv -dump-params

# Check all configuration
nextflow config main.nf -show-config
````

**Solutions:**

**1. Parameter typo:**
````bash
# ❌ Wrong
nextflow run main.nf --inpt test.csv

# ✅ Correct
nextflow run main.nf --input test.csv
````

**2. Check default values:**
````groovy
// nextflow.config
params {
    input = null  // No default
    outdir = 'results'  // Has default
}

// In main.nf, validate required params
if (!params.input) {
    error "Please provide --input parameter"
}
````

---

### Issue: Profile not being used

**Symptoms:**
Docker not enabled even though using -profile docker

**Debugging:**
````bash
# Check which profile is active
nextflow config -profile docker

# Check final configuration
nextflow run main.nf -profile docker -dump-hashes
````

**Solutions:**

**1. Profile name typo:**
````bash
# ❌ Wrong
nextflow run main.nf -profile dokcer

# ✅ Correct
nextflow run main.nf -profile docker
````

**2. Profile not defined:**
````groovy
// nextflow.config
profiles {
    docker {  // Must match -profile name exactly
        docker.enabled = true
    }
}
````

---

## Performance Optimization

### Issue: Too many processes running simultaneously

**Symptoms:**
System becomes unresponsive

**Solution:** Limit concurrent processes
````groovy
// nextflow.config
executor {
    queueSize = 10  // Max 10 jobs submitted at once
}

process {
    maxForks = 4  // Max 4 instances of each process
}

// Or per-process
process {
    withName: MEMORY_INTENSIVE {
        maxForks = 2  // Limit resource-heavy process
    }
}
````

---

### Issue: Pipeline doesn't use all available CPUs

**Symptoms:**
CPUs sitting idle while pipeline runs

**Solutions:**

**1. Increase parallelization:**
````groovy
executor.queueSize = 20  // Submit more jobs
````

**2. Check process concurrency:**
````groovy
// Make sure processes can run in parallel
workflow {
    // ✅ Good: Parallel
    FASTQC(reads_ch)  // All samples processed simultaneously
    
    // ❌ Bad: Sequential
    reads_ch.each { sample ->
        FASTQC(sample)  // One at a time
    }
}
````

---

## Getting Help

### 1. Check Nextflow Logs
````bash
# View recent runs
nextflow log

# Get details of a specific run
nextflow log <run_name> -f script,hash,name,status,exit,submit,duration

# View full log
cat .nextflow.log
````

### 2. Use Nextflow Tower (Optional)

Free monitoring service: https://tower.nf
````bash
nextflow run main.nf -with-tower
````

### 3. Community Resources

- **Nextflow Slack:** https://www.nextflow.io/slack-invite.html
- **GitHub Discussions:** https://github.com/nextflow-io/nextflow/discussions
- **Google Group:** https://groups.google.com/forum/#!forum/nextflow
- **Stack Overflow:** Tag `nextflow`

### 4. Report Issues

When reporting problems, include:
````bash
# Nextflow version
nextflow -version

# Full error message
nextflow run main.nf 2>&1 | tee error.log

# Configuration
nextflow config -show-config > config.txt

# System info
uname -a
docker --version  # or singularity --version
````

### 5. Minimal Reproducible Example

Create a minimal example that reproduces the problem:
````groovy
#!/usr/bin/env nextflow
nextflow.enable.dsl=2

// Minimal code that shows the problem
process TEST {
    script:
    """
    echo "This is where the problem occurs"
    """
}

workflow {
    TEST()
}
````

---

## Quick Reference: Common Fixes

| Problem | Quick Fix |
|---------|-----------|
| Process fails | Check `.command.err` in work directory |
| Out of memory | Increase memory: `memory '32 GB'` |
| Command not found | Add container or check PATH |
| Docker won't start | Start Docker Desktop / check service |
| Permission denied | Add `-u $(id -u):$(id -g)` to runOptions |
| Pipeline slow | Check timeline.html, increase parallelization |
| Channel empty | Use `.view()` to debug, check file patterns |
| Disk full | Clean work directory: `nextflow clean -f` |
| Resume doesn't work | Check if you changed process code |

---

## 🎯 Key Debugging Steps

1. **Read the error message** - It usually tells you exactly what's wrong
2. **Check the work directory** - Contains all execution details
3. **Use .view()** - Debug channel contents
4. **Test manually** - Run the command outside Nextflow
5. **Start simple** - Comment out parts to isolate the problem
6. **Check the docs** - https://nextflow.io/docs/latest/
7. **Ask for help** - Nextflow community is friendly!

---

## Emergency Commands
````bash
# Kill stuck pipeline
Ctrl+C

# Force kill
pkill -9 nextflow

# Clean everything
nextflow clean -f
rm -rf work/ .nextflow*

# Start fresh
nextflow run main.nf
````

---

**Need more help?** Join the [Nextflow Slack](https://www.nextflow.io/slack-invite.html)!