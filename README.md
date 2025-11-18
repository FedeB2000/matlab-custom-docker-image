# MATLAB Custom Docker Image

[![Docker](https://img.shields.io/badge/Docker-Ready-blue.svg)](https://www.docker.com/)
[![MATLAB](https://img.shields.io/badge/MATLAB-R2025b-orange.svg)](https://www.mathworks.com/)

Custom Docker image for MATLAB that allows you to install specific products, toolboxes, and support packages in an automated and reproducible way.

## 📋 Table of Contents

- [Overview](#-overview)
- [Prerequisites](#-prerequisites)
- [Configuration](#%EF%B8%8F-configuration)
- [Building the Image](#%EF%B8%8F-building-the-image)
- [Running the Container](#-running-the-container)
- [Package Management](#-package-management)
- [FAQ](#-faq)
- [Additional Resources](#-additional-resources)

---

## 🎯 Overview

This project allows you to create custom MATLAB Docker images with:
- **Configurable MATLAB release** (default: R2025b)
- **Selectable products and toolboxes** (over 150 options available)
- **Support packages** for hardware and external libraries
- **Update levels** (Update 0, 1, 2, etc.)
- **Non-root user** for enhanced security
- **Automated installation** via MathWorks Package Manager (mpm)

### Main Features

✅ Based on official `mathworks/matlab-deps` image  
✅ Support for all modern MATLAB releases  
✅ Modular installation of products and toolboxes  
✅ Configuration via comments in Dockerfile  
✅ Final size reduction through automatic cleanup  
✅ Compatible with Docker and Podman  

---

## 🔧 Prerequisites

### Required Software

- **Docker Desktop** (Windows/Mac) or **Docker Engine** (Linux)
  - Windows: https://docs.docker.com/desktop/install/windows-install/
  - Mac: https://docs.docker.com/desktop/install/mac-install/
  - Linux: https://docs.docker.com/desktop/install/linux-install/

  Verify installation in terminal:
  ```powershell
  # Verify Docker installation
  docker --version
  ```

- **MathWorks Account** with valid license
  - Required to download and activate MATLAB
  - Register at: https://www.mathworks.com/

### System Requirements

| Component | Minimum Requirement | Recommended |
|-----------|-------------------|--------------|
| RAM | 8 GB | 16 GB |
| Disk Space | 10 GB free | 20+ GB free |
| CPU | 2 cores | 4+ cores |
| Docker | 20.10+ | Latest |

---

## ⚙️ Configuration

### 1. Product Selection

Open the `Dockerfile` and **uncomment** (remove the `#`) the products you want to install:

```dockerfile
# EXAMPLE: To install MATLAB base and Computer Vision Toolbox
product.MATLAB                          # ← Uncomment this line
product.Computer_Vision_Toolbox         # ← Uncomment this line
#product.Deep_Learning_Toolbox          # ← Leave commented if not needed
```

### 2. Release Configuration

You can specify the MATLAB release in two ways:

**Option A - Build Argument** (recommended):
```powershell
docker build --build-arg MATLAB_RELEASE=R2024b -t matlab-custom:R2024b .
```

**Option B - Modify Dockerfile**:
```dockerfile
ARG MATLAB_RELEASE=R2024a  # Change the release here
```

### 3. Update Level

Configure the update level (0 = base release, 1 = Update 1, etc.):

**Option A - Build Argument** (recommended):
```powershell
docker build --build-arg UPDATELEVEL=1 -t matlab-custom:R2024b .
```

**Option B - Modify Dockerfile**:
```dockerfile
ARG UPDATELEVEL=1  # Change the update number
```

### 4. Installation Destination

**Option A - Build Argument** (recommended):
```powershell
docker build --build-arg DESTINATION_FOLDER="/usr/local/MATLAB/${MATLAB_RELEASE}" -t matlab-custom:R2024b .
```

**Option B - Modify Dockerfile**:
The default installation directory is:
```dockerfile
ARG DESTINATION_FOLDER="/usr/local/MATLAB/${MATLAB_RELEASE}"
```

### 5. Complete Configuration with Multiple Parameters

You can combine all parameters in a single build command:

```powershell
# Complete example: Release R2024b, Update 2, custom directory
docker build `
  --build-arg MATLAB_RELEASE=R2024b `
  --build-arg UPDATELEVEL=2 `
  --build-arg DESTINATION_FOLDER="/opt/matlab/R2024b" `
  -t matlab-custom:R2024b-u2-custom `
  .

# Example: Release R2023a, Update 0, default directory
docker build `
  --build-arg MATLAB_RELEASE=R2023a `
  --build-arg UPDATELEVEL=0 `
  -t matlab-custom:R2023a `
  .

# Example with verbose logging for debugging
docker build `
  --build-arg MATLAB_RELEASE=R2025b `
  --build-arg UPDATELEVEL=1 `
  --progress=plain `
  -t matlab-custom:R2025b-u1 `
  .
```

**Recommended tag naming conventions:**
- `matlab-custom:R2024b` - Base release without updates
- `matlab-custom:R2024b-u1` - Release with Update 1
- `matlab-custom:R2024b-u2-gpu` - With specific GPU support
- `matlab-custom:latest` - Latest stable version

**📝 Note on products:**  
Although it's technically possible to pass products via `--build-arg PRODUCTS="MATLAB Simulink Deep_Learning_Toolbox"`, this method is **not recommended** when you need to install many toolboxes. It's much more convenient and maintainable to directly modify the Dockerfile by uncommenting the desired products, especially if you have a long list of packages to install.

---

## 🏗️ Building the Image

### Basic Build

```powershell
# Navigate to the project directory with cd
# Build the image with custom tag
docker build -t matlab-custom:latest .
```

### Build with Custom Parameters

```powershell
# Specific release with update level
docker build `
  --build-arg MATLAB_RELEASE=R2024b `
  --build-arg UPDATELEVEL=2 `
  -t matlab-custom:R2024b-u2 `
  .

# With verbose logging
docker build -t matlab-custom:latest . --progress=plain
```

### Verify Build

```powershell
# List available images
docker images | Select-String "matlab"

# Inspect image details
docker inspect matlab-custom:latest
```

### Estimated Build Time

Times depend on internet connection speed (package downloads) and system resources:

| Configuration | Approximate Time |
|---------------|------------------|
| MATLAB base | 15-20 minutes |
| + 3-5 toolboxes | 25-35 minutes |
| + 10+ toolboxes | 45-60+ minutes |

**Note**: With slower connections, times can increase significantly.

---

## 🚀 Running the Container

### Interactive Mode (Recommended)

```powershell
# Start MATLAB in interactive mode (CPU only)
docker run -it --rm matlab-custom:latest -browser

# With CPU resource limitation
docker run -it --rm --cpus=4 --memory=8g matlab-custom:latest -browser

# With NVIDIA GPU support (requires NVIDIA Container Toolkit)
docker run -it --rm --gpus all matlab-custom:latest -browser

# With specific GPU (e.g., GPU 0)
docker run -it --rm --gpus device='0' matlab-custom:latest -browser

# With specific GPUs (e.g., GPU 0 and 3)
docker run -it --rm --gpus '"device=0,3"' matlab-custom:latest -browser

# With GPU and resource limitation
docker run -it --rm --gpus all --cpus=8 --memory=16g matlab-custom:latest -browser
```

**Docker Parameters:**
- `-it`: Interactive mode with terminal
- `--rm`: Automatically removes container on exit
- `--cpus=N`: Limits available CPU cores (e.g., 4)
- `--memory=Xg`: Limits available RAM (e.g., 8g)
- `--gpus all`: Enables ALL available GPUs
- `--gpus device=N`: Enables only specified GPU (e.g., device=0)
- `matlab-custom:latest`: Image name

### MATLAB Startup Options

MATLAB supports various execution modes via command-line arguments:

```powershell
# Batch mode (runs commands without GUI)
docker run -it --rm matlab-custom:latest -batch "disp('Hello MATLAB'); ver"

# Batch mode with GPU
docker run -it --rm --gpus all matlab-custom:latest -batch "g = gpuDevice; disp(g.Name)"

# Run specific script in batch
docker run -it --rm -v ${PWD}:/workspace -w /workspace matlab-custom:latest -batch "run('script.m')"

# Desktop mode (requires X11 forwarding)
docker run -it --rm -e DISPLAY=host.docker.internal:0 matlab-custom:latest -desktop

# Nodesktop mode (command-line interface)
docker run -it --rm matlab-custom:latest -nodesktop

# Nosplash mode (starts MATLAB without splash screen)
docker run -it --rm matlab-custom:latest -nosplash

# Nojvm mode (without Java Virtual Machine - lighter)
docker run -it --rm matlab-custom:latest -nojvm -batch "disp('Lightweight mode')"

# Limit computational threads
docker run -it --rm matlab-custom:latest -singleCompThread

# Verbose logging for debugging
docker run -it --rm matlab-custom:latest -logfile /tmp/matlab.log -batch "ver"
```

**Main MATLAB Options:**
- `-batch "commands"`: Executes MATLAB commands in non-interactive mode and exits
- `-r "commands"`: Executes MATLAB commands at startup (deprecated, use `-batch`)
- `-desktop`: Starts complete graphical interface (requires X11)
- `-nodesktop`: Starts without graphical interface (terminal only)
- `-nosplash`: Doesn't show splash screen at startup
- `-nojvm`: Starts without Java Virtual Machine (reduces memory, some functions unavailable)
- `-singleCompThread`: Limits MATLAB to single computational thread
- `-logfile <file>`: Specifies output log file
- `-sd <folder>`: Sets startup directory

**Complete MATLAB options documentation:**  
📖 https://www.mathworks.com/help/matlab/ref/matlabwindows.html

### Practical Examples with CPU and GPU

```powershell
# Run intensive computation on CPU with resource limitation
docker run -it --rm --cpus=8 --memory=16g matlab-custom:latest `
  -batch "A = rand(5000); B = A * A'; disp('Computation completed')"

# Run computation on GPU with monitoring
docker run -it --rm --gpus all matlab-custom:latest `
  -batch "g = gpuDevice; A = gpuArray(rand(5000)); B = A * A'; disp(['GPU: ' g.Name])"

# Batch script with GPU and directory mount
docker run -it --rm --gpus all `
  -v ${PWD}:/data `
  -w /data `
  matlab-custom:latest `
  -batch "data = load('input.mat'); result = processOnGPU(data); save('output.mat', 'result')"

# MATLAB desktop with GPU and X11 forwarding
docker run -it --rm --gpus all `
  -e DISPLAY=host.docker.internal:0 `
  -v ${PWD}:/workspace `
  matlab-custom:latest -desktop

# Parallel Computing with limited CPUs
docker run -it --rm --cpus=4 --memory=8g matlab-custom:latest `
  -batch "parpool('local', 4); parfor i=1:100, A(i)=sin(i); end; disp('Parallel done')"
```

### 🎮 GPU Configuration for Containers

To use NVIDIA GPUs in Docker containers, you need to install the **NVIDIA Container Toolkit**.

#### GPU Prerequisites
- **NVIDIA GPU** compatible with CUDA
- **NVIDIA Drivers** updated on host system
- **NVIDIA Container Toolkit** installed

#### NVIDIA Container Toolkit Installation

**Complete installation guide:**  
📖 https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html

#### Verify GPU Configuration

```powershell
# Verify NVIDIA driver on host system
nvidia-smi

# Test GPU in container
docker run --rm --gpus all matlab-custom:latest -batch "gpuDevice"

# Verify GPU information from MATLAB
docker run --rm --gpus all matlab-custom:latest -batch "disp(gpuDevice(1))"
```

---

## 📦 Package Management

### I Forgot to Install a Package!

#### Option 1: Rebuild the Image (Recommended)

**Step 1 - Stop and Remove Existing Containers**

```powershell
# List all containers (including stopped)
docker ps -a

# Stop running container
docker stop <CONTAINER_ID_or_NAME>

# Remove container
docker rm <CONTAINER_ID_or_NAME>

# Or in a single command (force removal)
docker rm -f <CONTAINER_ID_or_NAME>

# Remove ALL containers based on matlab-custom
docker ps -a --filter ancestor=matlab-custom:latest --format "{{.ID}}" | ForEach-Object { docker rm -f $_ }
```

**Step 2 - Modify the Dockerfile**

Uncomment the forgotten package:
```dockerfile
# Before (commented)
#product.Deep_Learning_Toolbox

# After (uncommented)
product.Deep_Learning_Toolbox
```

**Step 3 - Remove the Old Image**

```powershell
# Remove previous image
docker rmi matlab-custom:latest

# If you have multiple tags, remove them all
docker rmi $(docker images matlab-custom -q)

# Force removal if necessary
docker rmi -f matlab-custom:latest
```

**Step 4 - Rebuild the Image**

```powershell
# Rebuild WITHOUT cache (guarantees package installation)
docker build --no-cache -t matlab-custom:latest .

# Or with partial cache
docker build --pull -t matlab-custom:latest .
```

**Step 5 - Start the New Container**

```powershell
docker run -it --rm matlab-custom:latest
```

#### Option 2: Runtime Installation (Temporary)

If you need the package RIGHT NOW without rebuilding:

```powershell
# Start container as root
docker run -it --rm --user root matlab-custom:latest bash

# Inside the container, download mpm
cd /tmp
wget https://www.mathworks.com/mpm/glnxa64/mpm
chmod +x mpm

# Install missing package
./mpm install --release=R2025b --destination=/usr/local/MATLAB/R2025b --products Deep_Learning_Toolbox

# Start MATLAB
matlab
```

⚠️ **WARNING**: This method installs the package ONLY in the current container. It will be lost on restart!

#### Option 3: Commit Modified Container

```powershell
# After installing the package in the container
# (following Option 2)

# In another terminal, save container as new image
docker commit <CONTAINER_ID> matlab-custom:latest-updated

# Use the new image
docker run -it --rm matlab-custom:latest-updated
```

### Complete System Cleanup

```powershell
# Remove all stopped MATLAB containers
docker container prune -f

# Remove all unused MATLAB images
docker image prune -a -f

# Total cleanup (free disk space)
docker system prune -a --volumes -f

# Verify freed space
docker system df
```

---

## ❓ FAQ

### Q: Can I use this image in production?
**A**: Yes, but make sure to:
- Have valid licenses for all installed products
- Implement regular data backups
- Monitor resources and performance
- Follow corporate security policies

### Q: Can I use GPU in the container?
**A**: Yes, with Docker GPU support:
```powershell
docker run --gpus all -it --rm matlab-custom:latest
```
Requires NVIDIA Container Toolkit and NVIDIA Driver installed.

---

## 📚 Additional Resources

- [MATLAB Docker Documentation](https://github.com/mathworks-ref-arch/matlab-dockerfile)
- [MathWorks Package Manager (mpm)](https://github.com/mathworks-ref-arch/matlab-dockerfile/blob/main/MPM.md)
- [Docker Documentation](https://docs.docker.com/)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
- [MATLAB Container Images on Docker Hub](https://hub.docker.com/u/mathworks)

---

## ⚠️ Disclaimer

This Docker image does not include MATLAB licenses. You must have valid licenses to use MathWorks products. For licensing information, contact MathWorks.

---

**Created with ❤️ for the MATLAB community**