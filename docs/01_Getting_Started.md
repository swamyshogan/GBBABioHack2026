# Getting Started with Nextflow Hackathon

This guide will help you set up your environment for the hackathon.

---

## 📋 Prerequisites

- **Operating System:** Linux, macOS, or Windows (with WSL2)
- **RAM:** 8GB minimum (16GB recommended)
- **Disk Space:** 20GB free
- **Internet:** Required for downloads

---

## 🔧 Installation Steps

### 1. Install Nextflow

**Linux & macOS:**
```bash
# Install Nextflow
curl -s https://get.nextflow.io | bash

# Make it executable and move to PATH
chmod +x nextflow
sudo mv nextflow /usr/local/bin/

# Verify installation
nextflow -version
```

**Expected output:**
```
nextflow version 23.10.1.5891
```

**Windows (WSL2):**
```bash
# First install WSL2 and Ubuntu
# Then follow Linux instructions above
```

---

### 2. Install Docker

#### macOS
1. Download [Docker Desktop for Mac](https://www.docker.com/products/docker-desktop)
2. Install and start Docker Desktop
3. Verify: `docker --version`

#### Linux (Ubuntu/Debian)
```bash
# Install Docker
sudo apt-get update
sudo apt-get install docker.io

# Add user to docker group (to avoid sudo)
sudo usermod -aG docker $USER

# Log out and back in, then verify
docker --version
docker run hello-world
```

#### Windows
1. Download [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop)
2. Ensure WSL2 is enabled
3. Verify in WSL2: `docker --version`

---

### 3. Install Java (if not present)

Nextflow requires Java 11 or later.

**Check if Java is installed:**
```bash
java -version
```

**If not installed:**

**macOS:**
```bash
brew install openjdk@11
```

**Linux:**
```bash
sudo apt-get install openjdk-11-jdk
```

---

### 4. Install Git

**macOS:**
```bash
brew install git
```

**Linux:**
```bash
sudo apt-get install git
```

**Windows:**
Download from [git-scm.com](https://git-scm.com/)

---

### 5. Optional: Install Singularity (Alternative to Docker)

If you prefer Singularity (common on HPC systems):

**Linux:**
```bash
# Install dependencies
sudo apt-get install -y build-essential libssl-dev uuid-dev \
    libgpgme11-dev squashfs-tools libseccomp-dev wget pkg-config \
    git cryptsetup

# Install Go (required)
wget https://go.dev/dl/go1.21.0.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.21.0.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin

# Install Singularity
git clone https://github.com/sylabs/singularity.git
cd singularity
git checkout v3.11.4
./mconfig
cd builddir
make
sudo make install
```

---

## ✅ Verify Your Setup

Run this script to check everything:
```bash
#!/bin/bash

echo "=== Checking Nextflow Setup ==="

# Check Nextflow
if command -v nextflow &> /dev/null; then
    echo "✅ Nextflow installed: $(nextflow -version | head -n1)"
else
    echo "❌ Nextflow not found"
fi

# Check Docker
if command -v docker &> /dev/null; then
    echo "✅ Docker installed: $(docker --version)"
    if docker ps &> /dev/null; then
        echo "✅ Docker is running"
    else
        echo "❌ Docker is not running. Start Docker Desktop."
    fi
else
    echo "❌ Docker not found"
fi

# Check Java
if command -v java &> /dev/null; then
    echo "✅ Java installed: $(java -version 2>&1 | head -n1)"
else
    echo "❌ Java not found"
fi

# Check Git
if command -v git &> /dev/null; then
    echo "✅ Git installed: $(git --version)"
else
    echo "❌ Git not found"
fi

# Check disk space
echo ""
echo "=== Disk Space ==="
df -h . | awk 'NR==2 {print "Available: " $4}'

# Check memory
echo ""
echo "=== Memory ==="
free -h | awk 'NR==2 {print "Available: " $7}'

echo ""
echo "=== Setup Check Complete ==="
```

Save as `check_setup.sh`, then run:
```bash
chmod +x check_setup.sh
./check_setup.sh
```

---

## 🧪 Test Your Installation

### Test 1: Run Hello World
```bash
nextflow run hello
```

**Expected output:**
```
N E X T F L O W  ~  version 23.10.1
Launching `nextflow-io/hello` [cranky_gates] DSL2 - revision: 1d43adb819
executor >  local (4)
[6c/9e3a3e] process > sayHello (4) [100%] 4 of 4 ✔
Ciao world!
Bonjour world!
Hello world!
Hola world!
```

### Test 2: Pull a Bioinformatics Container
```bash
docker pull biocontainers/fastqc:v0.11.9_cv8
```

### Test 3: Run Example Pipeline
```bash
# Clone this repository
git clone https://github.com/[YOUR_REPO]/nextflow-hackathon-2025.git
cd nextflow-hackathon-2025/examples/minimal_pipeline

# Run the example
nextflow run main.nf
```

---

## 🐛 Troubleshooting

### Issue: "nextflow: command not found"

**Solution:**
```bash
# Check if nextflow is in PATH
echo $PATH

# Add to PATH (add to ~/.bashrc or ~/.zshrc)
export PATH=$PATH:/usr/local/bin

# Or move nextflow to a directory in PATH
sudo mv nextflow /usr/local/bin/
```

### Issue: Docker permission denied

**Solution:**
```bash
# Add user to docker group
sudo usermod -aG docker $USER

# Log out and log back in
# Or restart your terminal
```

### Issue: "Cannot connect to Docker daemon"

**Solution:**
- **macOS/Windows:** Start Docker Desktop
- **Linux:** `sudo systemctl start docker`

### Issue: Java not found

**Solution:**
```bash
# Install Java
sudo apt-get install openjdk-11-jdk  # Linux
brew install openjdk@11              # macOS
```

### Issue: Nextflow crashes with memory error

**Solution:**
```bash
# Increase Java memory
export NXF_OPTS='-Xms512m -Xmx2g'
```

---

## 🎓 Next Steps

Once your setup is working:

1. ✅ Read [Nextflow Basics](02_Nextflow_Basics.md)
2. ✅ Explore [Example Pipelines](../examples/)
3. ✅ Choose your [Task](../tasks/)
4. ✅ Download [Data](../data/README.md)

---

## 📞 Need Help?

- **Slack:** [SLACK CHANNEL]
- **Email:** [EMAIL]
- **Office Hours:** [TIMES]

---

**Ready? Let's learn Nextflow! →** [Nextflow Basics](02_Nextflow_Basics.md)