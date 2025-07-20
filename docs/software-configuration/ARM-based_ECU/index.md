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

### Cross-Compilation (Optional)

For faster builds, consider cross-compilation from an x86 host. See [Docker Cross-Compilation Guide](https://docs.docker.com/build/building/multi-platform/) for details.

### Containerized Development

For team collaboration and reproducible environments, see our [Containerized Development Guide](containerized-development.md).

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

<!-- ## Platform-Specific Customizations -->

<!-- For advanced ARM platform customizations including: -->

<!-- - Ansible automation for fleet deployment -->
<!-- - Hardware-specific optimizations -->
<!-- - Security hardening for production -->

<!-- See: [ARM Customization Guide](customization.md) -->

## Next Steps

See [Sensor Configuration Guide](../sensor-configuration/index.md) for detailed sensor setup
