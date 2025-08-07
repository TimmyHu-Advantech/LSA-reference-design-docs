# Source installation

## Prerequisites

- OS
  - [Ubuntu 22.04](https://releases.ubuntu.com/22.04/)

- ROS
  - [ROS 2 Humble](https://docs.ros.org/en/humble/index.html)

  For ROS 2 system dependencies, refer to [REP-2000](https://www.ros.org/reps/rep-2000.html).

- [Git](https://git-scm.com/)
  - [Registering SSH keys to GitHub](https://github.com/settings/keys) is preferable.

- Before proceeding, make sure your system is up to date and `Git` is installed:

   ```bash
   sudo apt-get -y update
   sudo apt-get -y install git
   ```
   
## How to set up a development environment

1. Clone `autowarefoundation/autoware` and move to the folder:

   ```bash
   git clone https://github.com/autowarefoundation/autoware.git
   cd autoware
   ```

2. If you are installing Autoware for the first time, you can automatically install the dependencies by using the provided Ansible script:

   ```bash
   ./setup-dev-env.sh
   ```

## How to set up a workspace

1. Create a workspace directory and a `src` folder inside it:

   ```bash
   cd autoware
   mkdir src
   ```
   Then, import the necessary packages using vcs:
   ```bash
   # Note: Make sure to run the following commands locally on your machine (not over remote SSH)
   vcs import src < autoware.repos  
   ```
   ```bash
   vcs import src < extra-packages.repos
   ```

2. Install dependent ROS packages

   Before building the workspace, make sure all required ROS dependencies are installed:

   ```bash
   source /opt/ros/humble/setup.bash
   sudo apt-get update && sudo apt-get upgrade
   rosdep update
   rosdep install -y --from-paths src --ignore-src --rosdistro $ROS_DISTRO
   ```

3. Install and setup ccache to speed up consecutive builds
(optional but recommended)

   3.1 Install Ccache:
   ```bash
   sudo apt-get install ccache
   ```
   3.2 Configure Ccache

   Create the Ccache configuration folder and file:
   ```bash
   mkdir -p ~/.cache/ccache
   touch ~/.cache/ccache/ccache.conf
   ```
   Set the maximum cache size. By default, it's `5 GB`, but you can increase it based on your system's available storage. In this example, we set it to `60 GB`:
   ```bash
   echo "max_size = 60G" >> ~/.cache/ccache/ccache.conf
   ```
   3.3 Integrate Ccache with your environment

   To ensure that Ccache is used during compilation, add the following lines to your `.bashrc`. This setup redirects `gcc` and `g++` calls through Ccache:
   ```bash
   export CC="/usr/lib/ccache/gcc"
   export CXX="/usr/lib/ccache/g++"
   export CCACHE_DIR="$HOME/.cache/ccache/" 
   ```
   After adding these lines, either reload your .bashrc or restart the terminal to apply the changes.

   3.4 Verify Ccache is working
   ```bash
   ccache -s
   ```

## How to build workspace

1. Create a new swapfile
(optional but recommended)

   Building Autoware is memory-intensive. If your system runs out of memory during the build, it may freeze or crash. To prevent this, it is recommended to configure 16–32 GB of swap space:

   ```bash
   sudo swapoff /swapfile
   sudo rm /swapfile
   ```
   Allocating `32 GB` of swap space improves build performance and helps prevent crashes due to memory exhaustion:
   ```bash
   sudo fallocate -l 32G /swapfile
   sudo chmod 600 /swapfile
   sudo mkswap /swapfile
   sudo swapon /swapfile
   ```

2. Build the workspace:
   
   Autoware uses [colcon](https://github.com/colcon) to build workspaces.
   For more advanced options, refer to the [documentation](https://colcon.readthedocs.io/).
   ```bash
   colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Release
   ```
   Reference:
   ```bash
      [debug]: "-DCMAKE_BUILD_TYPE=Debug"
      [min-size-rel]: "-DCMAKE_BUILD_TYPE=MinSizeRel"
      [rel-with-deb-info]: "-DCMAKE_BUILD_TYPE=RelWithDebInfo"
      [release]: "-DCMAKE_BUILD_TYPE=Release"     
   ```
   By reducing the number of packages built in parallel, you can lower memory usage. In the example below, only one package is built at a time:
   ```bash
   colcon build --symlink-install --parallel-workers 1 --cmake-args -DCMAKE_BUILD_TYPE=Release
   ```

## Autoware Build GUI

1. Install dependencies:

   ```bash
   sudo apt install libwebkit2gtk-4.1-0 libjavascriptcoregtk-4.1-0 libsoup-3.0-0 libsoup-3.0-common
   ```

   Before running the Autoware Build GUI, ensure that both `Rust` and `Node.js` are installed on your system:

2. Install Rust:

   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs/ | sh
   ```
   You can verify it using the following command:
   ```bash
   rustc --version
   ```
   You will see something similar to:
   ```console
   rustc 1.88.0 (6b00bc388 2025-06-23)
   ```

3. Install Node.js(version > 14):

   ```bash
   curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
   \. "$HOME/.nvm/nvm.sh"
   nvm install 22
   ```
   You can verify it using the following command:
   ```bash
   node --version
   ```
   You will see something similar to:
   ```console
   v22.17.0
   ```

4. It is recommended to install pnpm:

   ```bash
   npm install -g pnpm
   ```

5. Install gtk-3.0:

   ```bash
   sudo apt-get install libgtk-3-dev
   sudo apt-get install javascriptcoregtk-4.1
   sudo apt-get install libsoup-3.0
   sudo apt-get install webkit2gtk-4.1
   ```
  
6. Run the Build GUI from source:

   Clone the repository:
   ```bash
   git clone https://github.com/leo-drive/autoware-build-gui.git
   ```
   Navigate to the project directory:
   ```bash
   cd autoware-build-gui
   ```
   Install the required packages:
   ```bash
   pnpm i
   ```
   Run the Build GUI:
   ```bash
   pnpm tauri dev
   ```


## Network Configuration

Enable localhost-only communication
1. Enable multicast on lo

   Manually (temporary solution):

   ```bash
   sudo ip link set lo multicast on
   ```
   This will be reverted once the computer restarts. To make it permanent, follow the steps below:

   On startup with a service (permanent solution):
   ```bash
   sudo nano /etc/systemd/system/multicast-lo.service
   ```
   Paste the following into the file:
   ```service
   [Unit]
   Description=Enable Multicast on Loopback

   [Service]
   Type=oneshot
   ExecStart=/usr/sbin/ip link set lo multicast on

   [Install]
   WantedBy=multi-user.target
   ```
   Make it take effect:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable multicast-lo.service
   sudo systemctl start multicast-lo.service
   ```
   You can verify it using the following command:
   ```bash
   sudo systemctl status multicast-lo.service
   ```
   You will see something similar to:
   ```console
   ○ multicast-lo.service - Enable Multicast on Loopback
     Loaded: loaded (/etc/systemd/system/multicast-lo.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Thu 2025-07-31 06:41:35 EDT; 37min ago
    Process: 793 ExecStart=/usr/sbin/ip link set lo multicast on (code=exited, status=0/SUCCESS)
   Main PID: 793 (code=exited, status=0/SUCCESS)
        CPU: 2ms

   Jul 31 06:41:35 afer360-Default-string systemd[1]: Starting Enable Multicast on Loopback...
   Jul 31 06:41:35 afer360-Default-string systemd[1]: multicast-lo.service: Deactivated successfully.
   Jul 31 06:41:35 afer360-Default-string systemd[1]: Finished Enable Multicast on Loopback.
   ```
   To verify with another command:
   ```bash
   ip link show lo
   ```
   You will see something similar to:
   ```console
   1: lo: <LOOPBACK,MULTICAST,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
   ```

2. Tune DDS settings

   For CycloneDDS configuration:

   Save the following file as `~/cyclonedds.xml`:
   ```console
   <?xml version="1.0" encoding="UTF-8" ?>
   <CycloneDDS xmlns="https://cdds.io/config" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="https://cdds.io/config https://raw.githubusercontent.com/eclipse-cyclonedds/cyclonedds/master/etc/cyclonedds.xsd">
     <Domain Id="any">
       <General>
         <Interfaces>
           <NetworkInterface autodetermine="false" name="lo" priority="default" multicast="default" />
         </Interfaces>
         <AllowMulticast>default</AllowMulticast>
         <MaxMessageSize>65500B</MaxMessageSize>
       </General>
       <Internal>
         <SocketReceiveBufferSize min="10MB"/>
         <Watermarks>
           <WhcHigh>500kB</WhcHigh>
         </Watermarks>
       </Internal>
       <Discovery>
         <ParticipantIndex>auto</ParticipantIndex>
         <MaxAutoParticipantIndex>100</MaxAutoParticipantIndex>
       </Discovery>
     </Domain>
   </CycloneDDS>
   ```
   Then add the following lines to your `~/.bashrc` file:
   ```console
   export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp

   export CYCLONEDDS_URI=file://absolute/path/to/cyclonedds.xml
   # Replace `/absolute/path/to/cyclonedds.xml` with the actual path to the file.
   # Example: export CYCLONEDDS_URI=file:///home/user/cyclonedds.xml
   ```