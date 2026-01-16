# Nextflow Hackathon 2025 🧬

**Build a Reproducible Bioinformatics Pipeline in 24 Hours**

[![Nextflow](https://img.shields.io/badge/nextflow-%E2%89%A523.10.0-23aa62.svg)](https://www.nextflow.io/)
[![Docker](https://img.shields.io/badge/docker-required-2496ED.svg)](https://www.docker.com/)

---

## 📅 Event Information

- **Seminar:** January 22, 2025 | 4:00-6:00 PM EST
- **Hackathon Start:** January 22, 2025 | 6:00 PM EST
- **Working Period:** 24 hours (remote)
- **Presentations:** January 23, 2025 | 6:00 PM EST (in-person)
- **Location:** [INSERT LOCATION]

---

## 🎯 Quick Links

| Resource | Description |
|----------|-------------|
| [**Getting Started**](docs/01_Getting_Started.md) | Install Nextflow & Docker |
| [**Nextflow Basics**](docs/02_Nextflow_Basics.md) | Learn Nextflow fundamentals |
| [**Tasks**](tasks/) | Choose your challenge |
| [**Data Download**](data/README.md) | Get datasets |
| [**Examples**](examples/) | Starter code |
| [**Cheatsheet**](resources/cheatsheet.md) | Quick reference |

---

## 📋 Choose Your Task

### [Task 1: RNA-Seq Analysis](tasks/task1_rnaseq.md) ⭐ Beginner
Process RNA-seq data from raw reads to differential expression.
- **Dataset:** 6 samples, paired-end, human chr1
- **Size:** ~800 MB

### [Task 2: Variant Calling](tasks/task2_variants.md) ⭐⭐ Intermediate
Identify germline variants from whole genome sequencing.
- **Dataset:** 3 samples, 30x coverage, human chr20
- **Size:** ~1.2 GB

### [Task 3: Multi-Omics Integration](tasks/task3_multiomics.md) ⭐⭐⭐ Advanced
Integrate RNA-seq and methylation data.
- **Dataset:** 4 matched samples, human chr21
- **Size:** ~1.5 GB

---

## 🚀 Quick Start (5 Minutes)
```bash
# 1. Install Nextflow
curl -s https://get.nextflow.io | bash
sudo mv nextflow /usr/local/bin/

# 2. Install Docker
# macOS/Windows: Download Docker Desktop
# Linux: sudo apt-get install docker.io

# 3. Verify setup
nextflow -version
docker run hello-world

# 4. Test with example
git clone https://github.com/[YOUR_USERNAME]/nextflow-hackathon-2025.git
cd nextflow-hackathon-2025/examples/minimal_pipeline
nextflow run main.nf
```

---

## 📚 Documentation

### For Beginners
1. [Getting Started](docs/01_Getting_Started.md) - Installation
2. [Nextflow Basics](docs/02_Nextflow_Basics.md) - Core concepts
3. [Containers 101](docs/04_Containers.md) - Docker basics
4. [Minimal Example](examples/minimal_pipeline/) - Starter template

### For Advanced Users
1. [Advanced Nextflow](docs/03_Nextflow_Advanced.md) - Optimization
2. [Best Practices](docs/05_Best_Practices.md) - Code quality
3. [Module Examples](examples/module_examples/) - Reusable modules

### Reference
- [Cheatsheet](resources/cheatsheet.md) - Quick commands
- [Troubleshooting](docs/06_Troubleshooting.md) - Debug guide
- [Container Registry](resources/container_registry.md) - Available images

---

## ✅ Requirements

### Deliverables
- ✅ Working Nextflow pipeline (DSL2)
- ✅ All processes containerized
- ✅ 2+ configuration profiles
- ✅ Clear documentation
- ✅ Example results

### Scoring (100 points)
- **Code Quality** (25 pts) - Clean, documented code
- **Nextflow Best Practices** (30 pts) - Proper use of features
- **Scientific Validity** (25 pts) - Correct analysis
- **Presentation** (20 pts) - Clear communication

---

## 💾 Data Download

All datasets available at: **[GOOGLE DRIVE/ZENODO LINK]**
```bash
# Quick download
wget [LINK] -O hackathon_data.tar.gz
tar -xzf hackathon_data.tar.gz
```

See [data/README.md](data/README.md) for details.

---

## 📝 Submission

**Deadline:** January 23, 2025, 6:00 PM EST

Submit via: **[GOOGLE FORM LINK]**

**What to include:**
- GitHub repository URL (must be public)
- Team information
- Task chosen

---

## 🆘 Getting Help

- **Slack:** [SLACK WORKSPACE]
- **Email:** [ORGANIZER EMAIL]
- **Office Hours:** Available on Slack Jan 22-23

---

## 🎁 Prizes

- 🥇 **1st Place:** [PRIZE]
- 🥈 **2nd Place:** [PRIZE]
- 🥉 **3rd Place:** [PRIZE]
- 🎨 **Best Visualization:** [PRIZE]
- 💡 **Most Innovative:** [PRIZE]

---

## 📖 Learning Path

**Never used Nextflow?** Follow this path:
```
Day 1: Install & Setup (1 hour)
  ↓
Day 2-3: Nextflow Basics (2-3 hours)
  ↓
Day 4-5: Work through examples (2-3 hours)
  ↓
Day 6-7: Start building your pipeline
```

**Have Nextflow experience?**
- Jump straight to [task descriptions](tasks/)
- Review [best practices](docs/05_Best_Practices.md)
- Check out [advanced techniques](docs/03_Nextflow_Advanced.md)

---

## 🤝 Contributing

Found an error? Have a suggestion?
- Open an [issue](../../issues)
- Submit a [pull request](../../pulls)

---

## 📜 License

This repository is licensed under MIT License. See [LICENSE](LICENSE) for details.

---

## 🙏 Acknowledgments

- [Nextflow](https://www.nextflow.io/)
- [nf-core](https://nf-co.re/)
- [BioContainers](https://biocontainers.pro/)
- [Seqera Labs](https://seqera.io/)

---

**Questions? Reach out on Slack or email us!**

**Good luck! 🚀**

---

*Repository maintained by Shogan Sugumar Swamy (Graduate Biotechnology and Bioinformatics Association)*
