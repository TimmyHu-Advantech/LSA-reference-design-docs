# Software Configuration

This guide covers deploying Autoware on Low Speed Autonomy (LSA) vehicles.

## Deployment Workflow

1. **[Getting Started](getting-started/index.md)** - Prepare your base system
2. **Platform Configuration** - Choose your platform:
     - [x86\_64 ECUs](x86_64-based_ECU/index.md) for Intel/AMD systems
     - [ARM ECUs](ARM-based_ECU/index.md) for NVIDIA Jetson platforms
3. **[Sensor Configuration](sensor-configuration/index.md)** - Set up your sensors
4. **[Middleware Configuration](middleware-configuration/index.md)** - Optimize performance

## Platform-Specific Deployment

### x86\_64-based ECUs
Deploy containerized Autoware on Intel/AMD platforms. [View tested ECUs](../hardware-configuration/ECUs/x86_64ECUs/index.md).

### ARM-based ECUs  
Deploy on NVIDIA Jetson and other ARM platforms for low-power applications. [View tested ECUs](../hardware-configuration/ECUs/armECUs/index.md).

## Key Configuration Areas

### Sensor Configuration
Configure network interfaces, LiDAR sensors, cameras, CAN bus, GNSS/IMU, and time synchronization. [Learn more](sensor-configuration/index.md).

### Middleware Configuration  
Optimize ROS 2 middleware for LSA applications. Zenoh is recommended for wireless and cellular networks. [Learn more](middleware-configuration/index.md).

## Resources

- [Autoware Documentation](https://autoware.org/)
- [ROS 2 Documentation](https://docs.ros.org/)
- [NVIDIA Jetson Resources](https://developer.nvidia.com/embedded-computing)
