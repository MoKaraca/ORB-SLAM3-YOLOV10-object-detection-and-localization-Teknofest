

# ROS2 ORB SLAM3 LOCALIZATION & YOLOV10 OBJECT DETECTION

## Teknofest 2023 Ulaşımda Yapay Zeka (AI in Transportation) Competition

This repository contains the localization and object detection pipeline developed for the Teknofest 2023 Ulaşımda Yapay Zeka competition. The system combines omputer vision with visual SLAM to achieve real-time spatial awareness and object tracking.

Key contributions and features developed for the competition include:
* **Custom Object Detection:** Trained and evaluated custom YOLOv8 (and YOLOv10) models on a massive curated dataset of over 76,000 transportation images.
* **Extensive Model Optimization:** Conducted over 100 training experiments to incrementally refine model accuracy and performance, producing highly optimized weights.
* **Validation & Testing:** Implemented automated video slicing and inference testing scripts for continuous validation of model performance on real-world test footage.
* **ROS2 Integration:** Combined the YOLO vision pipeline with ORB-SLAM3 within a ROS2 Humble environment to provide simultaneous localization, mapping, and intelligent object tracking.


## ORB-SLAM3 VISUAL

https://github.com/user-attachments/assets/0840deb5-e9ea-4cd4-902f-52514500cda3

## YOLOV10 VISUAL

https://github.com/user-attachments/assets/2fee560c-55b2-4121-b7ff-a61e55d67e6f

## Recent Improvements & Customizations

I've made several critical improvements to the original package to make it more
easier to use with custom cameras and live streams of images:

* **Standardized ROS 2 Image Headers:** I completely removed the custom separate `/mono_py_driver/timestep_msg` topic. Timestamps are now natively embedded and extracted directly from the `sensor_msgs/msg/Image` header, adhering to ROS 2 best practices and preventing synchronization drift.
* **Smart Python Driver:** The `mono_driver_node.py` has been upgraded to automatically filter out non-image files (like `.json`).
* **Universal Dataset Support:** You no longer need to rename your custom images to massive nanosecond Unix timestamps. If the driver detects standard sequential filenames (like `frame_000000.jpg`), it will automatically generate perfectly spaced nanosecond timestamps for you (assuming a 30 FPS camera). This makes it universally compatible with both official EuRoC datasets and arbitrary custom image folders.


## Testing platforms

1. Ryzen 5 7500f, x86_64 bit architecture , Ubuntu 22.04 LTS (Jammy Jellyfish) and RO2 Humble Hawksbill (LTS)

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

Follow the steps below to create the ```ros2_test``` workspace, install dependencies and build the package. Note, the workspace must be named ```ros2_test``` due to a HARDCODED path in the python node.

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
cd ~/ros2_test/
source ./install/setup.bash
ros2 run ros2_orb_slam3 mono_node_cpp --ros-args -p node_name_arg:=mono_slam_cpp
```

In another terminal [python node]

```bash
cd ~/ros2_test
source ./install/setup.bash
ros2 run ros2_orb_slam3 mono_driver_node.py --ros-args -p settings_name:=MyCamera -p image_seq:=teknofest_img_dataset
```


## Credits & Acknowledgements

This repository is a customized fork built for my Teknofest project. 
All credit for the original ROS 2 wrapper implementation goes to **Azmyin Md. Kamal**. 
All credit for the core ORB-SLAM3 library goes to its original authors: Carlos Campos, Richard Elvira, Juan J. Gómez, José M. M. Montiel, and Juan D. Tardós (University of Zaragoza).

