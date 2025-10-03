# FAST-LIVO2_PX4-1.13_XTDrone_Gazebo

This project aims to build a simulation platform for FAST-LIVO2 based on PX4 1.13, XTDrone, and Gazebo.


## Setting up XTDrone environment with PX4 v1.13
Recommended environment:
Ubuntu 20.04, ROS Noetic

Configure XTDrone following the official documentation, which can be referenced here:

```bash
https://www.yuque.com/xtdrone/manual_cn/basic_config_13  
```

For ROS Noetic configuration, you can use FishROS's one-click setup script to simplify the deployment process:
One-click installation command:

```bash
wget http://fishros.com/install -O fishros && . fishros
```

## Livox Plugin Setup

```bash
cd ~/catkin_ws/src
# The version in this repository has already been modified with appropriate parameters and calibration; no extra steps needed.
git clone https://github.com/HeweiZhang2026/FAST-LIVO2_PX4-1.13_XTDrone_Gazebo  
cd ../
catkin build
source ~/catkin_ws/devel/setup.bash
```

According to the XTDrone documentation, you need to modify line 54 in `Livox-simulation-customMsg/src/livox_points_plugin.cpp` to point to your local CSV path. You can also change this path to select different Livox LiDAR models (e.g., Avia, Mid40, etc.).
Importantly, on line 101 of `src/livox_points_plugin.cpp`, you can set:
publishPointCloudType = 3;
to switch among three message types: PointCloud, PointCloud2, and CustomMsg, corresponding to parameters 1, 2, and 3 respectively.
Note that FAST-LIVO2 and FAST-LIO2 use CustomMsg by default.


## FAST-LIVO2 Environment Setup

Official MARS LAB repository:
https://github.com/hku-mars/FAST-LIVO2  

Requirements:

1. PCL && Eigen && OpenCV

PCL>=1.8, Follow PCL Installation.

Eigen>=3.3.4, Follow Eigen Installation.

OpenCV>=4.2, Follow Opencv Installation.

2. Sophus

Sophus Installation for the non-templated/double-only version.

git clone https://github.com/strasdat/Sophus.git  
cd ~/Sophus
git checkout a621ff
mkdir build && cd build && cmake ..
make
sudo make install

3. Vikit

```bash
cd catkin_ws/src
git clone https://github.com/xuankuzcr/rpg_vikit.git  
```

4. Build

Clone the repository and run catkin_make:

```bash
cd ~/catkin_ws/src
# The version in this repository has already been modified with appropriate parameters and calibration; no extra steps needed.
git clone https://github.com/HeweiZhang2026/FAST-LIVO2_PX4-1.13_XTDrone_Gazebo  
cd ../
catkin build
```


## Running
You can choose a custom launch file. This repository currently supports the `iris_realsense_livox` drone. It is recommended to modify the drone section as follows:

```bash
     <!-- iris_0 -->
     <group ns="iris_0">
        <!-- MAVROS and vehicle configs -->
            <arg name="ID" value="0"/>
            <arg name="ID_in_group" value="0"/>
            <arg name="fcu_url" default="udp://:24540@localhost:34580"/>
        <!-- PX4 SITL and vehicle spawn -->
        <include file="$(find px4)/launch/single_vehicle_spawn_xtd.launch">
            <arg name="x" value="0"/>
            <arg name="y" value="0"/>
            <arg name="z" value="0"/>
            <arg name="R" value="0"/>
            <arg name="P" value="0"/>
            <arg name="Y" value="0"/>
            <arg name="vehicle" value="iris"/>
            <arg name="sdf" value="iris_realsense_livox"/>
            <arg name="mavlink_udp_port" value="18570"/>
            <arg name="mavlink_tcp_port" value="4560"/>
            <arg name="ID" value="$(arg ID)"/>
            <arg name="ID_in_group" value="$(arg ID_in_group)"/>
        </include>
```

Run the following command to start Gazebo simulation:

```bash
roslaunch px4 outdoor_my.launch
```

Launch FAST-LIVO2:

```bash
roslaunch fast_livo mapping_avia.launch
```

## Tips:

If you want to switch drones/cameras, ensure the following:
1. The image resolution matches the camera model configuration.
2. Confirm the camera intrinsic parameters.

Taking `iris_realsense_livox` as an example, you can check the actual image dimensions using:

```bash
rostopic echo -n 1 /iris_0/stereo_camera/left/image_raw/width
rostopic echo -n 1 /iris_0/stereo_camera/left/image_raw/height
```

If you are unsure about the exact camera intrinsics, it's best to obtain the correct parameters from the Gazebo model or the camera publisher. You can retrieve complete camera information using:

```bash
rostopic echo -n 1 /iris_0/stereo_camera/left/camera_info
```

You need to modify the `camera_pinhole.yaml` file to match the actual camera parameters.

```bash
cam_model: Pinhole
cam_width: 752
cam_height: 480
scale: 1.0
cam_fx: 375.9986188774177
cam_fy: 375.9986188774177
cam_cx: 376.0
cam_cy: 240.0
cam_d0: -0.1
cam_d1: 0.01
cam_d2: 0.00005
cam_d3: -0.0001
```
