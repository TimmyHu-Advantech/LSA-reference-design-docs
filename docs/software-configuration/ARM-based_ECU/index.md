# ARM-based ECU Configuration

Deploy Autoware on ARM platforms, focusing on NVIDIA Jetson and AGX Orin for LSA vehicles.

## System Preparation

### CUDA Toolkit Installation

#### For NVIDIA AGX Orin

**Important**: Do NOT install CUDA packages manually on AGX Orin as it may break the system. CUDA is included with JetPack and must be installed through NVIDIA's SDK Manager.

**Installation Steps:**

1. **Download NVIDIA SDK Manager** from [https://developer.nvidia.com/nvidia-sdk-manager](https://developer.nvidia.com/nvidia-sdk-manager)

2. **Select JetPack 6.0** (or latest version) in SDK Manager
   - This will flash the Orin device and install the appropriate CUDA version
   - JetPack 6.0 includes CUDA 12.2, cuDNN, TensorRT, and other essential libraries

3. **Flash and Install**
   - Connect your AGX Orin to the host PC via USB-C
   - Follow SDK Manager prompts to flash the device
   - The process will install Ubuntu, CUDA, and all necessary drivers

4. **Verify Installation** after flashing:
```bash
# Check CUDA version
nvcc --version

# Verify Jetson platform and monitor system
sudo apt install -y python3-pip
pip3 install jetson-stats
sudo jtop

# Check JetPack version
cat /etc/nv_tegra_release
```

#### For Other ARM64 Platforms

For ARM64 platforms other than NVIDIA Jetson:
- Check the manufacturer's product specifications or manual for CUDA support
- Most non-NVIDIA ARM platforms do not support CUDA
- Consider using CPU-only or other acceleration options if CUDA is not available

**Supported JetPack Versions by Platform:**
- AGX Orin: JetPack 6.0 or later (recommended)
- Xavier Series: JetPack 5.1 or later
- Nano/TX2: Check NVIDIA's compatibility matrix

## Platform-Specific Optimizations

### NVIDIA AGX Orin Configuration

#### Power Management
Configure power modes based on deployment requirements:

```bash
# Development mode - Maximum performance
sudo nvpmodel -m 0  # MAXN mode
sudo jetson_clocks

# Production mode - Balanced performance/efficiency
sudo nvpmodel -m 1  # 30W mode
sudo jetson_clocks --restore
```

#### Memory Configuration
Optimize memory allocation for Autoware workloads:

```bash
# Increase GPU memory allocation
echo "gpu_mem_size=8G" | sudo tee /etc/modprobe.d/tegra.conf

# Configure swap for memory-intensive operations
sudo fallocate -l 16G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

### Hardware Acceleration Setup

#### Enable DLA (Deep Learning Accelerator)
```yaml
# ansible/roles/agx_orin_dla/tasks/main.yml
---
- name: Enable DLA cores
  lineinfile:
    path: /etc/environment
    line: "{{ item }}"
  loop:
    - 'CUDA_VISIBLE_DEVICES=0'
    - 'DLA_VISIBLE_DEVICES=0,1'
    - 'TF_ENABLE_TENSORRT_DLA=1'
```

## Performance Tuning

### CPU Governor Settings
```bash
# Set performance governor for all cores
for cpu in /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor; do
  echo performance | sudo tee $cpu
done

# Make persistent
echo 'GOVERNOR="performance"' | sudo tee /etc/default/cpufrequtils
```

### GPU Optimization
```yaml
# ansible/roles/gpu_optimization/tasks/main.yml
---
- name: Set GPU clock to maximum
  command: nvidia-smi -pm 1

- name: Configure GPU memory growth
  lineinfile:
    path: /etc/environment
    regexp: '^TF_FORCE_GPU_ALLOW_GROWTH='
    line: 'TF_FORCE_GPU_ALLOW_GROWTH=true'

- name: Set CUDA device order
  lineinfile:
    path: /etc/environment
    regexp: '^CUDA_DEVICE_ORDER='
    line: 'CUDA_DEVICE_ORDER=PCI_BUS_ID'
```

## Storage Optimization

### NVMe Configuration for High-Speed Logging

```yaml
# ansible/roles/storage_optimization/tasks/main.yml
---
- name: Configure NVMe for optimal performance
  lineinfile:
    path: /etc/fstab
    line: '/dev/nvme0n1p1 /var/log/autoware ext4 noatime,nodiratime,nobarrier 0 2'

- name: Set up log rotation for Autoware
  template:
    src: autoware_logrotate.j2
    dest: /etc/logrotate.d/autoware
```

### SD Card Optimization (Jetson Nano/Xavier NX)
```bash
# Reduce SD card wear
echo "vm.swappiness=10" | sudo tee -a /etc/sysctl.conf
echo "vm.vfs_cache_pressure=50" | sudo tee -a /etc/sysctl.conf

# Move temporary files to RAM
echo "tmpfs /tmp tmpfs defaults,noatime,mode=1777 0 0" | sudo tee -a /etc/fstab
```

## Monitoring and Debugging

### Jetson-Specific Monitoring

```bash
# Real-time system monitoring
sudo jtop

# GPU/CPU/Memory stats
tegrastats

# Temperature monitoring
cat /sys/devices/virtual/thermal/thermal_zone*/temp
```

### Performance Profiling
```bash
# Profile Autoware with Nsight Systems
nsys profile -t cuda,nvtx,osrt,cudnn,cublas \
  -o autoware_profile \
  ros2 launch autoware_launch autoware.launch.xml

# Analyze with Nsight Compute
ncu --target-processes all \
  --metrics gpu__time_duration.sum \
  ros2 run perception_node perception_node
```

## Next Steps

See [Sensor Configuration Guide](../sensor-configuration/index.md) for detailed sensor setup
