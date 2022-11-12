# File description
- orbslam3_docker_contents : raw codes from the docker container for ORB_SLAM
- CERTH_EAM_client_server/CERTH_EAM_client : code to execute client script in PC
- Note: docker image can be found the dockerhub [here](https://hub.docker.com/repository/docker/gdbk1124/orbslam3_docker_tcp), where all dependencies for executing python script exist. (Thus, we don't need to start from the scratch to set the environment up.)

## Issue to be solved

While executing the orbslam algorithm in the container, the following issue will occur:

> terminate called after throwing an instance of 'std::runtime_error'
>   what():  Pangolin X11: Failed to open X display


## Setting up automatically to build docker container

- cd orbslam3_docker

- ./build_container_slam.sh

Then, you can see the docker image is pulling from the dockerhub, and it builds the container from the image automatically. The bash script is to setting up the necessary commands for the container evnironment.

Note: To checkout the docker image is pulled in your local machine succesfully and if it is running, type the following:

- docker images

- docker ps

To get into the docker container created just now, type the following:

- **docker exec -it slam_container bash**



## Setting up manuelly for building docker container
0. Prerequisite: PC with ubuntu + realsense camera

1. In our local machine, pull the docker image by typing the following command in the terminal:
- **docker pull gdbk1124/orbslam3_docker_tcp:orbslam3**
2. While pulling the image, to make sure the realsense camera is connected to the PC, follow the [instructions](https://www.intelrealsense.com/get-started-depth-camera/) to start to setup.
3. After completing the pulling, we can now run the image as container :
- **docker run -d --name slam_container -it gdbk1124/orbslam3_docker_tcp:orbslam3**

where we tag the container with interepid and detach it while it runs in the background.

4. open another terminal and type:
- **git clone https://github.com/Kvasir8/ORB_SLAM3_Docker.git**
- **cd ORB_SLAM3_Docker/orbslam3_docker**
- **docker cp ORB_SLAM3/ slam_container:/**

In case that you want to get into the running container from the local machine, type the folllowing command:
- **docker exec -it slam_container bash**

where you can execute TCP server in docker container generated by the image

## How to start 

When starting docker container orblsam3 from docker image, make sure you're in the container. Execute the following commands in your local machine. (assumed that you already build docker container)

# Running orbslam3 without TCP connection

To play rosbag and feed it into ORB_SLAM algorithm, type the following commands in the container you created.

**docker start slam_container**

**docker exec -it slam_container bash**

Then, by the default according to bashrc, the starting working directory is set up as /ORB_SLAM3/Examples/ROS/ORB_SLAM3/CERTH_EAM_server, if the above instructions are done successfully.

Go to the ORB_SLAM3 directory by typing following command:
- **cd /ORB_SLAM3/**

open 3 tabs by following commands:

**roscore**

**rosbag play --pause /root/bagfiles/02_lowerResolution_compress.bag /device_0/sensor_1/Color_0/image/data:=/camera/rgb/image_raw /device_0/sensor_0/Depth_0/image/data:=/camera/depth_registered/image_raw**

**rosrun ORB_SLAM3 RGBD /ORB_SLAM3/Vocabulary/ORBvoc.txt /ORB_SLAM3/Examples/RGB-D/RealSense_D435i.yaml**

In case of the ORB_SLAM3 is failed to load the yaml file, execute the following command: 

**chmod +x build.sh**

**./build.sh**

(option)To test out ros images : ~/src/visual_slam/scripts$ python rosbag_play_test.py

Again, make sure you are in docker container, so that you can run ORB Slam algorithm in the docker container.

reference:

https://github.com/UZ-SLAMLab/ORB_SLAM3/tree/0df83dde1c85c7ab91a0d47de7a29685d046f637

https://github.com/RAFALAMAO/ORB_SLAM3_NOETIC

--

certh_server dependency issue in orbslam3 docker container:
http://wiki.ros.org/cv_bridge/Tutorials/ConvertingBetweenROSImagesAndOpenCVImagesPython

--

## How to test TCP connection with certh client and server
Make sure your local machine is connected to the realsense camera.
From client side (local machine):

1. Go to the directory: $cd CERTH_EAM_client_server/CERTH_EAM_client
2. execute python3 client script $python3 certh_client.py

From server side (docker container):
Assume you already made the container from the image

1. start docker container: $docker start orbslam3
2. goto container's bash: $docker exec -it orblsam3 bash
3. execute python3 server script by following working directory: /ORB_SLAM3/Examples/ROS/ORB_SLAM3/CERTH_EAM_server$python3 certh_server.py

- Note: Both client and server should be executed at the same time, so that we can avoid time out from the client side.

In case there's issue on displaying a window via Docker container, execute following command:
- **xhost +local:docker**

## Installing dependencies manuelly

In case that you want to build the image without image from Dockerhub, here is the following instrucitons:

As for the installation of realsense-ros, instruction can be followed by: https://github.com/IntelRealSense/realsense-ros

To brief, installation commends are followings:

For ddynamic_reconfigureConfig issue, follow instruction: **sudo apt-get install ros-noetic-ddynamic-reconfigure**

Also install realsense sdk 2.0: 
**sudo apt-get install librealsense2-dev**

**pip install pyrealsense2**



## Reference: 

- rosbag recording file [here](https://drive.google.com/drive/folders/1Ql-CqxX9xzk-axdcLj7F-jH8BA-1dO5b): 


- https://stackoverflow.com/questions/50935373/display-a-window-through-docker-container

- Intel RealSense ROS releases: https://dev.intelrealsense.com/docs/ros-wrapper

- how to remap ros topic in a launch file: https://roboticsbackend.com/ros-topic-remap-example/#In_a_launch_file

- realsense-ros setup instruction for ros-noetic, How to use RealSense with ubuntu 20.04 and ROS Noetic: https://linuxtut.com/en/be1eb78015828d43f9fb/

https://github.com/IntelRealSense/realsense-ros/issues/812

- RealSense_D435i.yaml: https://github.com/UZ-SLAMLab/ORB_SLAM3/tree/master/Examples/RGB-D

- ORB-SLAM3: https://github.com/UZ-SLAMLab/ORB_SLAM3/tree/0df83dde1c85c7ab91a0d47de7a29685d046f637

- Dataset: https://projects.asl.ethz.ch/datasets/doku.php?id=kmavvisualinertialdatasets#downloads

- Intelrealsense SDK : https://github.com/IntelRealSense/librealsense

- ros_numpy dependency issue : https://github.com/ros-gopigo/gopigo3_node/issues/6

etc: 

- Dockerhub login issue resolution: https://stackoverflow.com/questions/50151833/cannot-login-to-docker-account

- recording PC screen : https://obsproject.com/

- ssh key issue resolution: https://www.howtogeek.com/168119/fixing-warning-unprotected-private-key-file-on-linux/

- docker authentication issue while docker push: https://github.com/docker/docker-credential-helpers/issues/140
# ORB_SLAM3_Docker
