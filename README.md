# yolo_object_detector
Object detector in ROS using deep learning in Darknet - Yolo

Steps:
1) This package uses the proposed standard Vision messages, so clone/download the vision messages package and place it in the catkin workspace. Vision messages package: https://github.com/Kukanani/vision_msgs.git
2) Clone/Download this repository into your workspace which has two packages: `darknet_ros` and `dn_object_detect` 
3) RUN `catkin_make`
4) RUN `roslaunch dn_object_detect objdetect.launch` - (NOTE: This launch file also launches the openni2.launch file for the camera. If you are using any other camera, please change the camera topic in the launch file and uncomment the line that launches the openni camera)
