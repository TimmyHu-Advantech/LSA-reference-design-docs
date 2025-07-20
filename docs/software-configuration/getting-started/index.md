# Getting Started

Foundational setup for deploying Autoware on Ubuntu 22.04 (AMD64/ARM64).

## System Requirements

### Hardware Requirements

#### Minimum Specifications
- **CPU**: 8-core x86_64 (Intel/AMD) or ARMv8 (ARM64)
- **RAM**: 32 GB (16 GB for simulation-only)
- **Storage**: 30 GB of free space
- **GPU**: NVIDIA GPU with CUDA 11.8+ and Compute Capability 5.0+ (required for perception)

#### Recommended Specifications
- **CPU**: 16+ cores
- **RAM**: 64 GB+
- **Storage**: 100 GB+ SSD
- **GPU**: NVIDIA RTX 3080+ (x86_64) or AGX Orin (ARM)

### Network
- Gigabit Ethernet for sensors
- Internet access for setup
- Optional: Secondary interface for vehicle communication

### Operating System
- Ubuntu 22.04 LTS
- Kernel 5.15+
- Real-time kernel recommended for production

## System Preparation

### 1. Install Essential Packages

```bash
# Update package lists and upgrade system
sudo apt update && sudo apt upgrade -y

# Install essential tools
sudo apt install -y \
  build-essential \
  cmake \
  git \
  wget \
  curl \
  gnupg \
  lsb-release \
  software-properties-common \
  python3-pip \
  python3-venv
```

### 2. CUDA Toolkit Installation

CUDA installation is hardware-specific and varies significantly between x86 and ARM platforms. Please refer to the appropriate platform-specific guide for detailed CUDA installation instructions:

- **For x86_64-based ECUs**: See [x86\_64 CUDA Installation Guide](../x86_64-based_ECU/index.md#cuda-and-gpu-configuration)
- **For ARM-based ECUs**: See [ARM CUDA Installation Guide](../ARM-based_ECU/index.md#cuda-toolkit-installation)

## Environment Setup

The Autoware project provides an automated setup script that installs all necessary dependencies and configures your development environment.

### 1. Clone Autoware Repository

```bash
# Clone the Autoware repository
git clone https://github.com/autowarefoundation/autoware.git
cd autoware

# Checkout the 2025.02 branch
git checkout 2025.02
```

### 2. Run Setup Script

The setup script will automatically install ROS 2, development tools, and all required dependencies. The script may take 10-30 minutes to complete depending on your internet connection and system performance.


```bash
# Run the setup script
./setup-dev-env.sh
```

## Autoware Installation via Debian Packages

The NEWSLab team provides pre-built Debian packages for Autoware that simplify the installation process. This method is faster than building from source and ensures consistent deployments.

### 1. Configure Autoware Local Repository

Download and install the appropriate repository configuration package:

#### For x86\_64 Systems:
```bash
# Download the repository configuration package
wget https://github.com/NEWSLabNTU/autoware/releases/download/rosdebian%2F2025.02-1/autoware-localrepo_2025.2-1_amd64.deb

# Install the repository configuration
sudo apt install ./autoware-localrepo_2025.2-1_amd64.deb
```

#### For ARM64 Systems (Non-Jetson):
```bash
# Download the repository configuration package
wget https://github.com/NEWSLabNTU/autoware/releases/download/rosdebian%2F2025.02-1/autoware-localrepo_2025.2-1_arm64.deb

# Install the repository configuration
sudo apt install ./autoware-localrepo_2025.2-1_arm64.deb
```

#### For NVIDIA Jetson Platforms (AGX Orin with JetPack 6.0):
```bash
# Download the Jetson-specific repository configuration package
wget https://github.com/NEWSLabNTU/autoware/releases/download/rosdebian%2F2025.02-1/autoware-localrepo_2025.2-1_jetpack6.0.deb

# Install the repository configuration
sudo apt install ./autoware-localrepo_2025.2-1_jetpack6.0.deb
```

### 2. Install Autoware

Update apt package lists and install the full Autoware stack or specific components:

```bash
sudo apt update
sudo apt install -y autoware-full
```

### 3. Configure Environment

Set up your shell environment to use the installed Autoware:

```bash
# Add Autoware setup to bashrc
echo "source /opt/autoware/autoware-env" >> ~/.bashrc
source ~/.bashrc

# Verify installation
ros2 pkg list | grep autoware
```

**Note**: The Debian packages install Autoware to `/opt/autoware` and are designed to coexist with other ROS 2 installations.

## Post-Installation Verification

This section uses a shell script to verify if the Autoware is successfully deployed to the system.

### 1. System Check Script

Download the system check script [here](assets/verify-installation.sh) and move the shell script to the working directory. Executes the verification script to verify the deployment.

```bash
chmod +x verify-installation.sh
./verify-installation.sh
```

### 2. Test Basic Functionality

Follow the [tutorial](https://autowarefoundation.github.io/autoware-documentation/main/tutorials/ad-hoc-simulation/planning-simulation/) to run the planning simulation.

## Next Steps

With the base system configured, proceed to one of the following sections:

1. **[x86_64-based ECU Configuration](../x86_64-based_ECU/index.md)** for Intel/AMD platforms
2. **[ARM-based ECU Configuration](../ARM-based_ECU/index.md)** for NVIDIA Jetson and the other ARM platforms
