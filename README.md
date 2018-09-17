AIRCAP: Aerial Outdoor Motion Capture --  Public Code Repository
=================================================================

Project Webpage: [http://aircap.is.tuebingen.mpg.de](http://aircap.is.tuebingen.mpg.de)

NEWS:  Nodes and packages specific to our submission to RA-L + ICRA 2019 added. Please scroll below for details.

# Copyright and License

All Code in this repository - unless otherwise stated in local license or code headers is

Copyright 2018 Max Planck Institute for Intelligent Systems

Licensed under the terms of the GNU General Public Licence (GPL) v3 or higher.
See: https://www.gnu.org/licenses/gpl-3.0.en.html


# Contents:

ROS Packages:

* /packages/flight -- mandatory packages needed for/on aerial vehicles
* /packages/optional -- packages required for specific hardware (camera interface modules)
* /packages/replay -- packages needed to replay data on the ground
* /scrips/ -- start and stop scripts to run everything locally and remotely on real robots - see readme.txt in that directory
* /scrips/simulation -- start and stop scripts to run a swarm formation in gazebo simulation - needs additional dependencies


# Compiling
Link all flight and optional packages in catkin workspace on a flight capable node
build packages with catkin_make

##Requirements:
* [ROS] (http://wiki.ros.org/kinetic) 
* [LibrePilot ROS Interface] (https://github.com/AIRCAP/LibrePilot)
* [SSD Multibox Detection Server] (https://github.com/AIRCAP/caffe)
* [modified g2o] (https://github.com/aamirahmad/g2o)
* [fkie_multimaster] (http://wiki.ros.org/fkie_multimaster)
* [mav_msgs] (http://wiki.ros.org/mav_msgs)
* [mrpt_bridge] (http://wiki.ros.org/mrpt_bridge)
* [pose_cov_ops] (http://wiki.ros.org/pose_cov_op)
* [rviz_plugin_covariance] (http://wiki.ros.org/rviz_plugin_covariance) (optional, for rviz isualization)

##Additional Requirements for simulation:
* [Gazebo] (http://gazebosim.org/) -- tested with Gazebo 8.6
* [rotors_gazebo] (https://github.com/AIRCAP/rotors_gazebo) -- hint: add the entire tree to catkin_ws/src

# Setup

Flying vehicles are connected via Wifi. Each flying vehicle needs a main
computer running Linux and ROS (tested with kinetic).  Either the main computer
or a separate dedicated GPU needs to be running caffe with Wei Liu's SSD
Multibox detector and our ssd_server [repository]
(https://github.com/AIRCAP/caffe)

Alternatively a seperate GPU board can be connected via LAN (this is our setup.
We have SSD Multibox running on an NVIDIA Jetson TX1)

A Flight Controller running the LibrePilot firmware (suggested: OpenPilot
Revoltion) needs to be connected to the main computer via USB [repository]
(https://github.com/AIRCAP/LibrePilot)

A camera must be connected to the main computer and supply accurately
timestamped frames. Any camera with sufficiently high resolution and ROS
support can be used. We used BASLER and FLIR cameras and our own ROS nodes for
interfacing

## librepilot_node

The librepilot_node node from the librepilot package [separate repository]
(https://github.com/AIRCAP/LibrePilot) provides IMU and robot self pose data
and forwards waypoints to the flight controllers autopilot for aerial
navigation.

All Poses are in NED (north east down) coordinate frame.

## tf_from_uav_pose

This node provides static transformations between various frames such as camera
position relative to vehicle position, or NED world frame relative to ROS ENU
reference frame.


## model_distance_from_height

The projection_models model_distance_from_height node provides various
transformations required by other nodes to translate from camera to world frame
and back. This is responsible for translating 2d detections from the neural
network into 3d uncertainty ellipses (PoseWithCovarianceStamped) in world
frame, as well as translating 3d estimates into 2d coordinate region of
interests for the next detection.  You will have to modify launch files with
correct information regarding the placement and vehicle relative pose of the
camera on your vehicle.

## camera_configs

The node provided by this package is required by model_distance_from_height for
correct projections from camera into world frame and back.  You will have to
provide a config matching your camera. The camera should export under the topic
&lt;machine_namespace&gt;/video

## neural_network_detector

The neural network detector node is NOT running a neural network. It listens to
images and forwards them to a running ssd_server node - then exposes the
detections back into ROS as timestamped messages

## target_tracker_distributed_kf

The Kalman Filter runs on each vehicle, fuses the detections of all vehicles
(subscribed to using fkie_multimaster and robot specific namespaces) and
calculates the fused estimate which is published once every time the robot self
pose is updated from the flight controller It also tracks the offset between
the flight controller's GPS derived pose estimate and the actual pose as
estimated based on neural network detections.

## nmpc_planner 
This node corresponds to the work published in RA-L + IROS 2018. For our latest work on active perception see the next section and node.

An MPC based planner that allows the robots to follow the detected person.
WARNING! This code is in a very early state of development and not considered stable.
Like everything else use it at your own risk. You should always have means for manual override!!!
What the planner does is:
 * Listens to /machine_&lt;ID&gt;/pose for the robot is its running on
 * Listens to /machine_&lt;ID&gt;/poseThrottled for all other robots
 * Listens to /machine_&lt;ID&gt;/target_tracker/target for the current person location estimate.
 * Exports Waypoint to fly to into /machine_&lt;ID&gt;/command <br/>
Desired behaviour: All robots should keep a minimum distance from each other (implemented via potential field) and
at a given distance to the tracked person, at a given altitude.

## nmpc_planner mpc_act

Active Perception-driven convex MPC planner with integrated collision avoidance as submitted in manuscript to RA-L + ICRA 2019.

This package corresponds to the work submitted for review to RA-L + ICRA 2019. Packages other than nmpc_planner (see above) remain mostly the same.

# Additional Packages:

Some of these packages are specific to camera hardware, simulation environment, etc.

* /packages/flight/CMakeModules -- compilation helper macros
* /packages/flight/generic_potential_field -- used by nmpc_planner
* /packages/flight/pose_cov_ops_interface -- used by projection_models
* /packages/flight/uav_msgs -- ROS messages used by everything
* /packages/optional/basler_image_capture -- camera interface and video recording module for certain basler cameras
* /packages/optional/ptgrey_image_capture -- camara interface and video recording module for FLIR Blackfly cameras
* /packages/optional/gcsa_visualization -- Python scripts to create additional ROS messages for rviz visualization
* /packages/replay/video_replay_only -- replays AVI files and timestamp files created by the basler and ptgrey video modules back into ROS -- also includes a vidergrab node to record avi files from arbitrary sensor_msg/image streams in ROS with correct timestamps
* /packages/simulation/aircap -- launch files to start ros nodes for software in the loop simulation
* /packages/simulation/fake_communication_failure -- ROS node to simulate imperfect communication by dropping ROS messages
* /packages/simulation/librepilot_gazebo_bridge -- ROS node to simulate librepilot flight controller using gsimulated azebo firefly MAV
* /packages/simulation/random_moving_target -- ROS node to simulate a person in the simulation environment


