![License](https://img.shields.io/badge/License-GPLv3-blue.svg)
![Build Status](https://img.shields.io/badge/Build-Passing-success.svg)
![ROS2](https://img.shields.io/badge/ROS2-Humble-blue.svg)
![Version](https://img.shields.io/badge/Version-2.0.0-blue.svg)

# ROS2 ORB SLAM3 V1.0 package (Teknofest Edition)

A ROS2 package for ORB SLAM3 V1.0. Focus is on native integration with the ROS2 ecosystem. This version has been adapted and personalized for my Teknofest project (ORB-SLAM3 + YOLOv10 Object Detection and Localization).

My goal is to provide a solid, reliable starting point for using the ORB SLAM3 framework in ROS 2. This repository builds upon the original work by Azmyin Md. Kamal and the excellent ROS1 port of ORB SLAM3 by [thien94](https://github.com/thien94/orb_slam3_ros/tree/master). 

## 🚀 Recent Improvements & Customizations

We've made several critical improvements to the original package to make it more robust and easier to use with custom cameras and live streams:

* **Standardized ROS 2 Image Headers:** We completely removed the custom separate `/mono_py_driver/timestep_msg` topic. Timestamps are now natively embedded and extracted directly from the `sensor_msgs/msg/Image` header, adhering to ROS 2 best practices and preventing synchronization drift.
* **Smart Python Driver:** The `mono_driver_node.py` has been upgraded to automatically filter out non-image files (like `.json`).
* **Universal Dataset Support:** You no longer need to rename your custom images to massive nanosecond Unix timestamps! If the driver detects standard sequential filenames (like `frame_000000.jpg`), it will automatically generate perfectly spaced nanosecond timestamps for you (assuming a 30 FPS camera). This makes it universally compatible with both official EuRoC datasets and arbitrary custom image folders.

## 0. Preamble

* This package builds [ORB-SLAM3](https://github.com/UZ-SLAMLab/ORB_SLAM3) V1.0 as a shared internal library. Comes included with a number of Thirdparty libraries [DBoW2, g2o, Sophus]
* g2o used is an older version and is incompatible with the latest release found here [g2o github page](https://github.com/RainerKuemmerle/g2o).
* This package differs from other ROS1 wrappers, thien94`s ROS 1 port and ROS 2 wrappers in GitHub by supprting/adopting the following
  * A separate python node to send data to the ORB-SLAM3 cpp node. This is purely a design choice.
  * At least C++17 and Cmake>=3.8
  * Eigen 3.3.0, OpenCV 4.2, latest release of Pangolin
* Comes with a small test image sequence from EuRoC MAV dataset (MH05) to quickly test installation
* For newcomers in ROS2 ecosystem, this package serves as an example of building a shared cpp library and also a package with both cpp and python nodes.
* May not build or work correctly in **resource constrainted hardwares** such as Raspberry Pi 4, Jetson Nano

## Testing platforms

1. Intel i5-9300H, x86_64 bit architecture , Ubuntu 22.04 LTS (Jammy Jellyfish) and RO2 Humble Hawksbill (LTS)
2. AMD Ryzen 5600X, x86_64 bit architecture, Ubuntu 22.04 LTS (Jammy Jellyfish) and RO2 Humble Hawksbill (LTS)

## 1. Prerequisitis

Start with installing the following prerequisits

### Eigen3

```
sudo apt install libeigen3-dev
```

### Pangolin and configuring dynamic library path

We install Pangolin system wide and configure the dynamic library path so the necessary .so from Pangolin can be found by ros2 package during run time. More info here https://robotics.stackexchange.com/questions/105973/ros2-port-of-orb-slam3-can-copy-libdow2-so-and-libg2o-so-using-cmake-but-gettin

#### Install Pangolin

```
cd ~/Documents
git clone https://github.com/stevenlovegrove/Pangolin
cd Pangolin
./scripts/install_prerequisites.sh --dry-run recommended # Check what recommended softwares needs to be installed
./scripts/install_prerequisites.sh recommended # Install recommended dependencies
cmake -B build
cmake --build build -j4
sudo cmake --install build
```

#### Configure dynamic library

Check if ```/usr/lib/local``` is in the LIBRARY PATH

```bash
echo $LD_LIBRARY_PATH
```

If not, then perform the following 

```bash
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/lib/local
sudo ldconfig
```

Then open the ```.bashrc``` file in ```\home``` directory and add these lines at the very end

```bash
if [[ ":$LD_LIBRARY_PATH:" != *":/usr/local/lib:"* ]]; then
    export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
fi
```

Finally, source ```.bashrc``` file 

```bash
source ~/.bashrc
```
 
### OpenCV
Ubuntu 22.04 by default comes with >OpenCV 4.2. Check to make sure you have at least 4.2 installed. Run the following in a terminal

```bash
python3 -c "import cv2; print(cv2.__version__)" 
```

## 2. Installation

Follow the steps below to create the ```ros2_test``` workspace, install dependencies and build the package. Note, the workspace must be named ```ros2_test``` due to a HARDCODED path in the python node. I leave it to the developers to change this behavior as they see fit.

```bash
cd ~
mkdir -p ~/ros2_test/src
cd ~/ros2_test/src
git clone https://github.com/Mechazo11/ros2_orb_slam3.git
cd .. # make sure you are in ~/ros2_ws root directory
rosdep install -r --from-paths src --ignore-src -y --rosdistro humble
source /opt/ros/humble/setup.bash
colcon build --symlink-install
```

## 3. Monocular Example:

Run the builtin example to verify the package is working correctly
In one terminal [cpp node]

```bash
cd ~/ros2_ws/
source ./install/setup.bash
ros2 run ros2_orb_slam3 mono_node_cpp --ros-args -p node_name_arg:=mono_slam_cpp
```

In another terminal [python node]

```bash
cd ~/ros2_ws
source ./install/setup.bash
ros2 run ros2_orb_slam3 mono_driver_node.py --ros-args -p settings_name:=EuRoC -p image_seq:=sample_euroc_MH05
```

Both nodes would perform a handshake and the VSLAM framework would then work as shown in the following video clip


https://github.com/Mechazo11/ros2_orb_slam3/assets/44814419/af9eaa79-da4b-4405-a4d7-e09242ab9660


## Credits & Acknowledgements

This repository is a customized fork built for my Teknofest project. 
All credit for the original ROS 2 wrapper implementation goes to **Azmyin Md. Kamal**. 
All credit for the core ORB-SLAM3 library goes to its original authors: Carlos Campos, Richard Elvira, Juan J. Gómez, José M. M. Montiel, and Juan D. Tardós (University of Zaragoza).

## TODO next version:

- [ ] Stereo mode example
- [ ] RGBD mode example
- [ ] Detailed build instructions for `aarch64` based computers i.e. Orin Nano, Raspberry Pi etc.
