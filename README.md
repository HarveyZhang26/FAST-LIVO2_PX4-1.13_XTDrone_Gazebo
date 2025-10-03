# FAST-LIVO2_PX4-1.13_XTDrone_Gazebo

此项目旨在为FAST-LIVO2搭建基于PX4 1.13、XTDrone、Gazebo环境的仿真平台。


# PX4 1.13版XTDrone环境搭建
环境推荐：
Ubuntu 20.04，ROS Noetic

按照官方的使用文档配置XTDrone，可以参照以下文档：

https://www.yuque.com/xtdrone/manual_cn/basic_config_13

其中，ROS Noetic的配置可以使用鱼香ROS的一键配置脚本，简化部署流程：
一键安装指令：
wget http://fishros.com/install -O fishros && . fishros


# Livox插件搭建

cd ~/catkin_ws/src
# 使用本仓库的版本已做好参数修改和标定，无需额外操作。
git clone https://github.com/HeweiZhang2026/FAST-LIVO2_PX4-1.13_XTDrone_Gazebo
cd ../
catkin build
source ~/catkin_ws/devel/setup.bash

根据XTDrone使用文档，您需要更改Livox-simulation-customMsg的src/livox_points_plugin.cpp的54行csv路径为本地路径，也可以通过更改它来选择不同的livox系列雷达（即选择Avia，Mid40等型号）。
重要的是，您可以在src/livox_points_plugin.cpp的101行：
publishPointCloudType = 3;
以更改三种msg类型: PointCloud, PointCloud2 和 CustomMsg。其分别对应参数1,2,3.
其中，FAST-LIVO2和FAST-LIO2默认使用CustomMsg。


# FAST-LIVO2环境搭建

MARS LAB官方仓库：
https://github.com/hku-mars/FAST-LIVO2

Requirments:
1.PCL && Eigen && OpenCV
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

3.Vikit

cd catkin_ws/src
git clone https://github.com/xuankuzcr/rpg_vikit.git 

4. Build
Clone the repository and catkin_make:

cd ~/catkin_ws/src
# 使用本仓库的版本已做好参数修改和标定，无需额外操作。
git clone https://github.com/HeweiZhang2026/FAST-LIVO2_PX4-1.13_XTDrone_Gazebo
cd ../
catkin build


# 运行
您可以选择自定义的launch文件，本仓库目前适配了iris_realsense_livox无人机，建议将无人机段改为：

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

运行以启动gazebo仿真：

roslaunch px4 outdoor_my.launch

启动FAST-LIVO2：

roslaunch fast_livo mapping_avia.launch

# Tips:

如果想要更换无人机/相机，要保证：
1、图像的尺寸与相机模型配置的尺寸匹配
2、确定相机内参

以iris_realsense_livox为例，您可以通过该命令检查实际图像的尺寸：

rostopic echo -n 1 /iris_0/stereo_camera/left/image_raw/width
rostopic echo -n 1 /iris_0/stereo_camera/left/image_raw/height

如果你不确定准确的相机内参，最好从Gazebo模型或相机发布方获取正确的参数。你可以使用以下命令获取完整的相机信息：

rostopic echo -n 1 /iris_0/stereo_camera/left/camera_info

我们需要修改camera_pinhole.yaml文件来匹配实际的相机参数。

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
