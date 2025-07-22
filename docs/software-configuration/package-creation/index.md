# Debian Package Creation

This guide covers creating Debian packages for Autoware components, enabling easy deployment and version management.

## Overview

Creating Debian packages from ROS 2 packages provides:
- Version control and dependency management
- Fast deployment without compilation
- Consistent installations across systems
- Easy rollback capabilities

## Prerequisites

Install required tools:

```bash
sudo apt update
sudo apt install -y \
  dpkg-dev \
  debhelper \
  devscripts \
  equivs \
  python3-bloom \
  python3-rosdep \
  fakeroot
```

## Creating Debian Packages

### Single Package Creation

Create a Debian package from a single ROS 2 package:

```bash
#!/bin/bash
# make-deb.sh - Create .deb from ROS 2 package

PACKAGE_NAME=$1
WORKSPACE_PATH=${2:-$(pwd)}

# Source ROS environment
source /opt/ros/humble/setup.bash

# Navigate to package
cd $WORKSPACE_PATH/src/$PACKAGE_NAME

# Generate debian files using bloom
bloom-generate rosdebian --os-name ubuntu --os-version jammy --ros-distro humble

# Build the package
fakeroot debian/rules binary

# Move generated .deb files
mkdir -p ~/debs
mv ../*.deb ~/debs/

echo "Package created: ~/debs/${PACKAGE_NAME}_*.deb"
```

### Batch Package Creation

Process multiple packages in a workspace:

```bash
#!/bin/bash
# package-deb.sh - Batch create .deb packages

WORKSPACE_PATH=${1:-$(pwd)}
OUTPUT_DIR=${2:-~/autoware-debs}

mkdir -p $OUTPUT_DIR

# Find all packages with package.xml
for pkg in $(find $WORKSPACE_PATH/src -name package.xml -exec dirname {} \;); do
    PACKAGE_NAME=$(basename $pkg)
    echo "Processing: $PACKAGE_NAME"
    
    cd $pkg
    
    # Skip if already processed
    if [ -f debian/changelog ]; then
        echo "  Skipping - debian files already exist"
        continue
    fi
    
    # Generate debian files
    bloom-generate rosdebian \
        --os-name ubuntu \
        --os-version jammy \
        --ros-distro humble \
        || continue
    
    # Build package
    DEB_BUILD_OPTIONS=nocheck fakeroot debian/rules binary
    
    # Move results
    mv ../*.deb $OUTPUT_DIR/ 2>/dev/null || true
done

echo "Packages created in: $OUTPUT_DIR"
```

## Advanced Package Configuration

### Custom Control File

Create custom package metadata:

```bash
# debian/control
Source: autoware-custom-sensor-driver
Section: misc
Priority: optional
Maintainer: Your Name <your.email@example.com>
Build-Depends: debhelper (>= 9),
               ros-humble-ros-core,
               ros-humble-sensor-msgs,
               ros-humble-std-msgs
Standards-Version: 3.9.8
Homepage: https://github.com/your-org/custom-driver

Package: autoware-custom-sensor-driver
Architecture: any
Depends: ${shlibs:Depends},
         ${misc:Depends},
         ros-humble-ros-core,
         ros-humble-sensor-msgs,
         ros-humble-std-msgs
Description: Custom sensor driver for Autoware
 This package provides ROS 2 drivers for custom sensors
 used in Autoware autonomous driving stack.
```

### Post-Installation Scripts

Add post-install configuration:

```bash
# debian/postinst
#!/bin/bash
set -e

case "$1" in
    configure)
        # Create necessary directories
        mkdir -p /opt/autoware/config
        mkdir -p /var/log/autoware
        
        # Set permissions
        chmod 755 /opt/autoware/config
        
        # Configure systemd service if needed
        if [ -f /opt/autoware/systemd/autoware.service ]; then
            systemctl daemon-reload
            systemctl enable autoware.service
        fi
        
        # Update ROS environment
        echo "source /opt/autoware/setup.bash" >> /etc/bash.bashrc
        ;;
    
    abort-upgrade|abort-remove|abort-deconfigure)
        ;;
    
    *)
        echo "postinst called with unknown argument '$1'" >&2
        exit 1
        ;;
esac

exit 0
```

## Repository Management

### Creating Local APT Repository

Set up a local repository for your packages:

```bash
#!/bin/bash
# setup-local-repo.sh

REPO_BASE=~/local-apt-repo
DIST=jammy
COMPONENT=main
ARCH=arm64

# Create repository structure
mkdir -p $REPO_BASE/{dists/$DIST/$COMPONENT/binary-$ARCH,pool/$COMPONENT}

# Copy packages
cp ~/autoware-debs/*.deb $REPO_BASE/pool/$COMPONENT/

# Generate package index
cd $REPO_BASE
dpkg-scanpackages pool/$COMPONENT /dev/null | \
    gzip -9c > dists/$DIST/$COMPONENT/binary-$ARCH/Packages.gz

# Create Release file
cat > dists/$DIST/Release << EOF
Origin: Autoware Local Repository
Label: Autoware Local
Suite: $DIST
Codename: $DIST
Version: 1.0
Architectures: $ARCH
Components: $COMPONENT
Description: Local Autoware Package Repository
EOF

# Add repository to sources
echo "deb [trusted=yes] file://$REPO_BASE $DIST $COMPONENT" | \
    sudo tee /etc/apt/sources.list.d/autoware-local.list

sudo apt update
```

### Publishing to Remote Repository

Upload packages to a remote APT repository:

```bash
#!/bin/bash
# publish-packages.sh

REMOTE_HOST=repo.example.com
REMOTE_PATH=/var/www/apt/autoware
PACKAGES_DIR=~/autoware-debs

# Upload packages
rsync -avz --progress \
    $PACKAGES_DIR/*.deb \
    $REMOTE_HOST:$REMOTE_PATH/pool/main/

# Update remote repository
ssh $REMOTE_HOST << 'EOF'
cd /var/www/apt/autoware
dpkg-scanpackages pool/main /dev/null | \
    gzip -9c > dists/jammy/main/binary-arm64/Packages.gz
apt-ftparchive release dists/jammy > dists/jammy/Release
EOF
```

## Package Versioning

### Semantic Versioning

Follow semantic versioning for packages:

```bash
# Update version in package.xml
<version>1.2.3</version>

# Update debian changelog
dch -v 1.2.3-1 "New upstream release"
dch -a "Added feature X"
dch -a "Fixed bug Y"
```

### Automated Version Management

```python
#!/usr/bin/env python3
# update_version.py

import xml.etree.ElementTree as ET
import subprocess
import sys

def update_package_version(package_path, new_version):
    # Update package.xml
    package_xml = f"{package_path}/package.xml"
    tree = ET.parse(package_xml)
    root = tree.getroot()
    
    version_elem = root.find('version')
    version_elem.text = new_version
    
    tree.write(package_xml)
    
    # Update debian changelog
    subprocess.run([
        'dch', '-v', f'{new_version}-1',
        '-D', 'jammy',
        f'Update to version {new_version}'
    ], cwd=package_path)

if __name__ == "__main__":
    update_package_version(sys.argv[1], sys.argv[2])
```

## Multi-Architecture Builds

### Cross-Compilation Setup

Build packages for different architectures:

```bash
# Install cross-compilation tools
sudo apt install -y \
  gcc-aarch64-linux-gnu \
  g++-aarch64-linux-gnu \
  qemu-user-static

# Configure for ARM64 build on x86_64
export CC=aarch64-linux-gnu-gcc
export CXX=aarch64-linux-gnu-g++
export CMAKE_TOOLCHAIN_FILE=/opt/ros/humble/share/ros_cross_compile/cmake/toolchains/aarch64.cmake
```

### Docker-Based Multi-Arch Build

```dockerfile
# Dockerfile.multiarch
ARG ARCH=arm64
FROM --platform=linux/${ARCH} ros:humble-ros-base

# Install build dependencies
RUN apt-get update && apt-get install -y \
    dpkg-dev \
    debhelper \
    python3-bloom \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /workspace

# Copy source
COPY src/ ./src/

# Build script
COPY scripts/build_packages.sh /
RUN chmod +x /build_packages.sh

# Build packages
RUN /build_packages.sh

# Extract packages
FROM scratch
COPY --from=0 /workspace/debs/* /
```

Build for multiple architectures:

```bash
# Build for ARM64
docker buildx build --platform linux/arm64 \
    --output type=local,dest=./packages/arm64 \
    -f Dockerfile.multiarch .

# Build for AMD64
docker buildx build --platform linux/amd64 \
    --output type=local,dest=./packages/amd64 \
    -f Dockerfile.multiarch .
```

## Package Testing

### Installation Testing

```bash
#!/bin/bash
# test-package.sh

PACKAGE_FILE=$1
TEST_CONTAINER="autoware-package-test"

# Create test container
docker run -d --name $TEST_CONTAINER \
    ubuntu:22.04 tail -f /dev/null

# Copy and install package
docker cp $PACKAGE_FILE $TEST_CONTAINER:/tmp/
docker exec $TEST_CONTAINER bash -c "
    apt update
    apt install -y /tmp/$(basename $PACKAGE_FILE)
    # Run basic tests
    dpkg -L $(basename $PACKAGE_FILE .deb)
    # Check for missing dependencies
    apt-get check
"

# Cleanup
docker rm -f $TEST_CONTAINER
```

### Integration Testing

```yaml
# .gitlab-ci.yml snippet
test-packages:
  stage: test
  image: ros:humble
  script:
    - apt update
    - apt install -y ./packages/*.deb
    - source /opt/ros/humble/setup.bash
    - ros2 pkg list | grep autoware
    - ros2 run autoware_package test_node
```

## Troubleshooting

### Common Issues

1. **Missing Dependencies**
   ```bash
   # Check dependencies
   dpkg-deb -I package.deb | grep Depends
   
   # Install missing dependencies
   sudo apt-get install -f
   ```

2. **Build Failures**
   ```bash
   # Clean build environment
   debian/rules clean
   
   # Verbose build
   DEB_BUILD_OPTIONS="nostrip noopt" fakeroot debian/rules binary
   ```

3. **Version Conflicts**
   ```bash
   # Force version
   sudo dpkg -i --force-overwrite package.deb
   
   # Check conflicts
   apt-cache policy package-name
   ```

## Best Practices

1. **Dependency Management**: Specify exact versions for critical dependencies
2. **Testing**: Always test packages in clean environments
3. **Documentation**: Include README.Debian with package-specific notes
4. **Signing**: Sign packages for production repositories
5. **Automation**: Use CI/CD for consistent package builds
6. **Cleanup**: Remove build artifacts and temporary files