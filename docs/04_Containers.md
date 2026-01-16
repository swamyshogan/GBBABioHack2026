# Containers for Reproducibility

Learn how to use Docker and Singularity containers in Nextflow pipelines for complete reproducibility.

---

## 📖 Table of Contents

1. [Why Containers?](#why-containers)
2. [Docker Basics](#docker-basics)
3. [Using Containers in Nextflow](#using-containers-in-nextflow)
4. [Finding Bioinformatics Containers](#finding-bioinformatics-containers)
5. [Singularity Alternative](#singularity-alternative)
6. [Best Practices](#best-practices)
7. [Troubleshooting](#troubleshooting)

---

## Why Containers?

### The Problem: "It Works on My Machine"
```bash
# Without containers - Environment-dependent
#!/bin/bash
# Requires: Python 3.9, bwa 0.7.17, samtools 1.15
# What if user has different versions?
# What if tools aren't installed?
```

### The Solution: Containers
```groovy
// With containers - Always works
process ALIGN {
    container 'biocontainers/bwa:0.7.17--h7132678_9'
    
    script:
    """
    # Exact same environment every time
    # No installation needed
    # Works on any system
    """
}
```

### Benefits

- ✅ **Reproducibility:** Same results every time
- ✅ **Portability:** Run anywhere (laptop, HPC, cloud)
- ✅ **No installation:** Tools pre-packaged
- ✅ **Version control:** Exact tool versions specified
- ✅ **Isolation:** No conflicts between tools

---

## Docker Basics

### What is Docker?

Docker packages software and its dependencies into a "container image" that can run anywhere.
```
┌─────────────────────────────────┐
│        Your Application         │
├─────────────────────────────────┤
│    Libraries & Dependencies     │
├─────────────────────────────────┤
│         Container Runtime       │
├─────────────────────────────────┤
│        Host Operating System    │
└─────────────────────────────────┘
```

### Installing Docker

**macOS/Windows:**
- Download [Docker Desktop](https://www.docker.com/products/docker-desktop)

**Linux (Ubuntu/Debian):**
```bash
sudo apt-get update
sudo apt-get install docker.io
sudo usermod -aG docker $USER  # Add user to docker group
# Log out and back in
```

### Basic Docker Commands
```bash
# Pull an image from registry
docker pull ubuntu:20.04

# List downloaded images
docker images

# Run a command in a container
docker run ubuntu:20.04 echo "Hello from container!"

# Interactive shell
docker run -it ubuntu:20.04 /bin/bash

# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# Remove a container
docker rm <container_id>

# Remove an image
docker rmi ubuntu:20.04

# Clean up everything
docker system prune -a
```

### Testing a Bioinformatics Container
```bash
# Pull FastQC container
docker pull biocontainers/fastqc:v0.11.9_cv8

# Run FastQC help
docker run biocontainers/fastqc:v0.11.9_cv8 fastqc --help

# Run on actual file (mount current directory)
docker run -v $PWD:$PWD -w $PWD biocontainers/fastqc:v0.11.9_cv8 \\
    fastqc sample.fastq.gz
```

---

## Using Containers in Nextflow

### Method 1: Per-Process Container
```groovy
process FASTQC {
    container 'biocontainers/fastqc:v0.11.9_cv8'
    
    input:
    path fastq
    
    output:
    path "*.html"
    
    script:
    """
    fastqc ${fastq}
    """
}
```

### Method 2: Global Container
```groovy
// nextflow.config
process.container = 'biocontainers/fastqc:v0.11.9_cv8'

docker.enabled = true
```

### Method 3: Profile-Based
```groovy
// nextflow.config
profiles {
    docker {
        docker.enabled = true
        process.container = 'biocontainers/multi-tool:latest'
    }
    
    singularity {
        singularity.enabled = true
        process.container = 'docker://biocontainers/multi-tool:latest'
    }
}
```

**Usage:**
```bash
nextflow run main.nf -profile docker
```

### Complete Example

**main.nf:**
```groovy
#!/usr/bin/env nextflow
nextflow.enable.dsl=2

process FASTQC {
    container 'biocontainers/fastqc:v0.11.9_cv8'
    
    input:
    path fastq
    
    output:
    path "*_fastqc.{html,zip}"
    
    script:
    """
    fastqc -q ${fastq}
    """
}

process MULTIQC {
    container 'ewels/multiqc:v1.14'
    publishDir 'results', mode: 'copy'
    
    input:
    path '*'
    
    output:
    path 'multiqc_report.html'
    
    script:
    """
    multiqc .
    """
}

workflow {
    reads_ch = Channel.fromPath('data/*.fastq.gz')
    FASTQC(reads_ch)
    MULTIQC(FASTQC.out.collect())
}
```

**nextflow.config:**
```groovy
docker {
    enabled = true
    runOptions = '-u $(id -u):$(id -g)'  // Run as current user
}
```

**Run:**
```bash
nextflow run main.nf
```

---

## Finding Bioinformatics Containers

### 1. BioContainers

**Website:** https://biocontainers.pro/

Search for tools, find Docker/Singularity images.

**Example searches:**
```bash
# FastQC
biocontainers/fastqc:v0.11.9_cv8

# BWA
biocontainers/bwa:0.7.17--h7132678_9

# SAMtools
biocontainers/samtools:1.16.1--h6899075_1

# STAR
quay.io/biocontainers/star:2.7.10b--h9ee0642_0
```

### 2. Quay.io

**Website:** https://quay.io/organization/biocontainers

Direct container registry, searchable.
```bash
docker pull quay.io/biocontainers/fastqc:0.11.9--0
```

### 3. Docker Hub

**Website:** https://hub.docker.com/

General container registry.
```bash
docker pull ubuntu:20.04
docker pull continuumio/miniconda3
```

### 4. nf-core Modules

**Website:** https://nf-co.re/modules

Pre-built Nextflow modules with tested containers.
```groovy
// Example from nf-core
include { FASTQC } from './modules/nf-core/fastqc/main'
```

---

## Container Best Practices

### 1. Always Pin Versions
```groovy
// ❌ Bad - version can change
container 'biocontainers/fastqc:latest'

// ✅ Good - specific version
container 'biocontainers/fastqc:v0.11.9_cv8'

// ✅ Even better - with digest
container 'biocontainers/fastqc@sha256:abc123...'
```

### 2. Document Container Sources
```groovy
process FASTQC {
    // Container: biocontainers/fastqc
    // Version: 0.11.9
    // Source: https://quay.io/repository/biocontainers/fastqc
    container 'biocontainers/fastqc:v0.11.9_cv8'
    
    script:
    """
    fastqc ...
    """
}
```

### 3. Test Containers Before Using
```bash
# Test the container works
docker run biocontainers/fastqc:v0.11.9_cv8 fastqc --version

# Test with real data
docker run -v $PWD:$PWD -w $PWD \\
    biocontainers/fastqc:v0.11.9_cv8 \\
    fastqc test.fastq.gz
```

### 4. Use Multi-Tool Containers Carefully
```groovy
// ❌ Avoid when possible - hard to version control
container 'bioinformatics/all-tools:latest'

// ✅ Prefer single-tool containers
process FASTQC {
    container 'biocontainers/fastqc:v0.11.9_cv8'
}

process MULTIQC {
    container 'ewels/multiqc:v1.14'
}
```

### 5. Handle File Permissions
```groovy
// nextflow.config
docker {
    enabled = true
    runOptions = '-u $(id -u):$(id -g)'  // Run as current user, not root
}
```

---

## Singularity Alternative

### Why Singularity?

- Used on many HPC systems (doesn't require root)
- Can run Docker containers
- Better for shared systems

### Installing Singularity

**Linux (Ubuntu):**
```bash
sudo apt-get install singularity-container
```

**Or use Conda:**
```bash
conda install -c conda-forge singularity
```

### Using Singularity in Nextflow

**nextflow.config:**
```groovy
profiles {
    singularity {
        singularity.enabled = true
        singularity.autoMounts = true
    }
}
```

**Run:**
```bash
nextflow run main.nf -profile singularity
```

### Converting Docker to Singularity
```bash
# Nextflow does this automatically, or manually:
singularity pull docker://biocontainers/fastqc:v0.11.9_cv8
```

### Singularity in Process
```groovy
process FASTQC {
    container 'docker://biocontainers/fastqc:v0.11.9_cv8'
    // Singularity will pull from Docker Hub
    
    script:
    """
    fastqc ...
    """
}
```

---

## Docker vs Singularity Comparison

| Feature | Docker | Singularity |
|---------|--------|-------------|
| **Root required** | Yes (desktop), No (rootless) | No |
| **HPC friendly** | Limited | Yes |
| **Image format** | Layers | Single file |
| **Running images** | Docker Hub, Quay.io | Docker images + SIF files |
| **Nextflow support** | Excellent | Excellent |
| **Best for** | Local development | HPC clusters |

---

## Common Container Registries

### 1. Quay.io (Recommended for Bioinformatics)
```groovy
container 'quay.io/biocontainers/fastqc:0.11.9--0'
```

### 2. Docker Hub
```groovy
container 'ubuntu:20.04'
container 'continuumio/miniconda3:latest'
```

### 3. GitHub Container Registry
```groovy
container 'ghcr.io/organization/tool:v1.0'
```

---

## Creating Custom Containers (Advanced)

### Dockerfile Example
```dockerfile
FROM ubuntu:20.04

# Install dependencies
RUN apt-get update && apt-get install -y \\
    wget \\
    python3 \\
    python3-pip

# Install tool
RUN pip3 install cutadapt==4.1

# Set working directory
WORKDIR /data

# Default command
CMD ["cutadapt", "--version"]
```

**Build:**
```bash
docker build -t my-cutadapt:1.0 .
```

**Test:**
```bash
docker run my-cutadapt:1.0
```

---

## Troubleshooting

### Issue 1: Permission Denied
```bash
# Error: Permission denied when accessing files

# Solution: Run as current user
docker {
    runOptions = '-u $(id -u):$(id -g)'
}
```

### Issue 2: Cannot Connect to Docker Daemon
```bash
# Error: Cannot connect to the Docker daemon

# Solution 1: Start Docker Desktop (macOS/Windows)

# Solution 2: Add user to docker group (Linux)
sudo usermod -aG docker $USER
# Log out and back in

# Solution 3: Check Docker is running
sudo systemctl status docker
```

### Issue 3: Container Not Found
```bash
# Error: Unable to pull image

# Solution: Check spelling and availability
docker search fastqc
docker pull biocontainers/fastqc:v0.11.9_cv8
```

### Issue 4: Out of Disk Space
```bash
# Error: No space left on device

# Solution: Clean up Docker
docker system prune -a
docker volume prune
```

### Issue 5: Slow Container Pulls
```bash
# Issue: Large containers taking long to download

# Solution 1: Use a closer registry mirror
# Solution 2: Pull in advance
docker pull biocontainers/fastqc:v0.11.9_cv8

# Solution 3: Cache containers locally
singularity.cacheDir = '/path/to/cache'
```

---

## Pre-Pulling Containers for Hackathon
```bash
#!/bin/bash
# pre_pull_containers.sh

echo "Pulling common bioinformatics containers..."

# Quality Control
docker pull biocontainers/fastqc:v0.11.9_cv8
docker pull ewels/multiqc:v1.14

# Alignment
docker pull quay.io/biocontainers/bwa:0.7.17--h7132678_9
docker pull quay.io/biocontainers/star:2.7.10b--h9ee0642_0

# Processing
docker pull quay.io/biocontainers/samtools:1.16.1--h6899075_1

# Quantification
docker pull quay.io/biocontainers/subread:2.0.1--h5bf99c6_1

# Differential expression
docker pull quay.io/biocontainers/bioconductor-deseq2:1.36.0--r41h9f5acd7_0

echo "Container pull complete!"
```

---

## 🎯 Key Takeaways

1. **Containers ensure reproducibility** - Same environment every time
2. **Always pin versions** - Don't use `:latest`
3. **BioContainers is your friend** - Pre-built bioinformatics tools
4. **Docker for local, Singularity for HPC** - Use appropriate tool
5. **Test containers before using** - Verify they work

---

## 🧪 Practice Exercises

### Exercise 1: Run FastQC in Docker
Pull the FastQC container and run it on a sample file.

### Exercise 2: Create a Multi-Container Pipeline
Build a pipeline using 3 different containers.

### Exercise 3: Switch Between Docker and Singularity
Create profiles that allow running with either container engine.

---

## 📚 Next Steps

- ✅ [Best Practices](05_Best_Practices.md)
- ✅ [Troubleshooting](06_Troubleshooting.md)

---

**Ready for best practices?** → [Best Practices](05_Best_Practices.md)