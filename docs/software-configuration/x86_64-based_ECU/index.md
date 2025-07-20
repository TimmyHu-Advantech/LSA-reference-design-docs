# x86_64-based ECU Configuration

Deploy Autoware on Intel/AMD platforms for LSA vehicles.

## CUDA and GPU Configuration

### 1. CUDA Toolkit Installation

For x86-based systems, CUDA must be installed manually. Visit the [NVIDIA CUDA Toolkit Archive](https://developer.nvidia.com/cuda-toolkit-archive) to find the appropriate version for your system.

**Recommended Installation Method: deb (network)**

Here's an example of installing CUDA Toolkit 12.3 using the network deb method:

```bash
# Step 1: Download and install the cuda-keyring package
# Visit https://developer.nvidia.com/cuda-toolkit-archive and select:
# - Operating System: Linux
# - Architecture: x86_64
# - Distribution: Ubuntu
# - Version: 22.04
# - Installer Type: deb (network)

# Install the keyring
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb

# Step 2: Update package lists
sudo apt update

# Step 3: Install CUDA Toolkit
sudo apt install -y cuda-toolkit-12-3

# Step 4: Verify installation
nvidia-smi
nvcc --version
```

### 2. Install Additional GPU Libraries

```bash
# Install cuDNN (after CUDA toolkit is installed)
# Visit https://developer.nvidia.com/cudnn for the latest version
sudo apt install -y cudnn

# Install TensorRT
sudo apt install -y tensorrt
```

### 3. Configure GPU Environment

```bash
# Add CUDA to PATH
echo 'export PATH=/usr/local/cuda/bin:$PATH' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH' >> ~/.bashrc
source ~/.bashrc

# Verify GPU is accessible
nvidia-smi
```

## Performance Monitoring

### System Monitoring Tools

```bash
# CPU and system monitoring
htop

# GPU monitoring
nvtop

# ROS 2 specific monitoring
ros2 run rqt_top rqt_top

# Network monitoring
iftop
```

### ROS 2 Performance Analysis

```bash
# Monitor topic frequencies
ros2 topic hz /sensing/lidar/concatenated/pointcloud

# Check node CPU usage
ros2 run rqt_top rqt_top

# Analyze communication graph
ros2 run rqt_graph rqt_graph
```

## Next Steps

See [Sensor Configuration Guide](../sensor-configuration/index.md) for detailed sensor setup.
