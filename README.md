### SDCND Capstone Project
This is the project repo for the final project of the Udacity Self-Driving Car Nanodegree: Programming a Real Self-Driving Car. For more information about the project, see the project introduction [here](https://classroom.udacity.com/nanodegrees/nd013/parts/6047fe34-d93c-4f50-8336-b70ef10cb4b2/modules/e1a23b06-329a-4684-a717-ad476f0d8dff/lessons/462c933d-9f24-42d3-8bdc-a08a5fc866e4/concepts/5ab4b122-83e6-436d-850f-9f4d26627fd9).

### Team members:

| Names               | Email Addresses          |
| -------------       |:-------------:           |
| Saimir Baci (TL)    | saimirbaci@hotmail.com   |
| Selçuk Çavdar       | cavdarselcuk@gmail.com   |
| Kadir Haspalamutgil | kadirhas@sabanciuniv.edu |
| Adithya Ranga       | aranga@umich.edu         |
| Dibakar Sigdel      | sigdeldkr@gmail.com      |

### Installation and Setup:
Please use **one** of the two installation options, either native **or** docker installation.

### Native Installation

* Be sure that your workstation is running Ubuntu 16.04 Xenial Xerus or Ubuntu 14.04 Trusty Tahir. [Ubuntu downloads can be found here](https://www.ubuntu.com/download/desktop).
* If using a Virtual Machine to install Ubuntu, use the following configuration as minimum:
  * 2 CPU
  * 2 GB system memory
  * 25 GB of free hard drive space

  The Udacity provided virtual machine has ROS and Dataspeed DBW already installed, so you can skip the next two steps if you are using this.

* Follow these instructions to install ROS
  * [ROS Kinetic](http://wiki.ros.org/kinetic/Installation/Ubuntu) if you have Ubuntu 16.04.
  * [ROS Indigo](http://wiki.ros.org/indigo/Installation/Ubuntu) if you have Ubuntu 14.04.
* [Dataspeed DBW](https://bitbucket.org/DataspeedInc/dbw_mkz_ros)
  * Use this option to install the SDK on a workstation that already has ROS installed: [One Line SDK Install (binary)](https://bitbucket.org/DataspeedInc/dbw_mkz_ros/src/81e63fcc335d7b64139d7482017d6a97b405e250/ROS_SETUP.md?fileviewer=file-view-default)
* Download the [Udacity Simulator](https://github.com/udacity/CarND-Capstone/releases).

### Docker Installation
[Install Docker](https://docs.docker.com/engine/installation/)

Build the docker container
```bash
docker build . -t capstone
```

Run the docker file
```bash
docker run -p 4567:4567 -v $PWD:/capstone -v /tmp/log:/root/.ros/ --rm -it capstone
```

### Port Forwarding
To set up port forwarding, please refer to the [instructions from term 2](https://classroom.udacity.com/nanodegrees/nd013/parts/40f38239-66b6-46ec-ae68-03afd8a601c8/modules/0949fca6-b379-42af-a919-ee50aa304e6a/lessons/f758c44c-5e40-4e01-93b5-1a82aa4e044f/concepts/16cf4a78-4fc7-49e1-8621-3450ca938b77)

### Usage

1. Clone the project repository
```bash
git clone https://github.com/udacity/CarND-Capstone.git
```

2. Install python dependencies
```bash
cd CarND-Capstone
pip install -r requirements.txt
```
3. Make and run styx
```bash
cd ros
catkin_make
source devel/setup.sh
roslaunch launch/styx.launch
```
4. Run the simulator

### Real world testing
1. Download [training bag](https://s3-us-west-1.amazonaws.com/udacity-selfdrivingcar/traffic_light_bag_file.zip) that was recorded on the Udacity self-driving car.
2. Unzip the file
```bash
unzip traffic_light_bag_file.zip
```
3. Play the bag file
```bash
rosbag play -l traffic_light_bag_file/traffic_light_training.bag
```
4. Launch your project in site mode
```bash
cd CarND-Capstone/ros
roslaunch launch/site.launch
```
5. Confirm that traffic light detection works on real life images
=======

### Implementation Details:

#### Waypoint Updater:
Planning part of this capstone comprises of the waypoint loader and updater nodes.
The waypoint updater node publishes the next 100 waypoints in front of the car, maintaining a target velocity and
constantly updating at 35 Hz.
- The updater node has base waypoints, current pose of the ego vehicle and traffic light waypoint information as inputs
and it publishes the final waypoints.
- We calculate the closest waypoint index from all the waypoints using the KDTree search logic.
Base waypoints from this closest index to the farthest given the number of lookahead points are calculated.
- Traffic light detector node determines closest traffic light and its state.
If we have a red signal the waypoint velocities are updated. decelerate_wps function calculates the distance from
the base waypoints to stop line and velocity is adjusted as function of distance and maximum deceleration rate.  

#### Drive by Wire (DBW) node:
DBW node is responsible of controlling the vehicle to follow the given trajectory. This node receives the current states of the vehicle and publishes steering, throttle and brake commands if there is no interruption from the driver. For the throttle, a PI controller is used, with a maximum throttle limitation to avoid sudden accelerations. The brake is controlled according to necessary torque to provide desired deceleration, which is estimated using vehicle's parameters. Steering is calculated according to desired angular velocity, which is given by a pure pursuit algorithm in the waypoint follower node. The controller loop is running at 50 Hz.
#### Traffic light detection:

The node subscribes image and traffic lights messages, and publishes the waypoints and the traffic light state as topics. For the detection of the traffic lights we are training a network based on the SSD architecture. SSD architecture has some advantages since it detects in one stage, and has low inference overhead. We used transfer learning to train the network. For this we used the Tensorflow Object Detection API, and pretrained network on the COCO dataset. The next part was to define the dataset for training the network, and currently we used a dataset that was build from previous students, for both scenarios, in the simulator and the udacity site. But what we noticed in the simulator scenario, it was not good at detecting the lights from far away, for this we annotated some more data for it, which helped improve the quality of the traffic light detector. The next part was to compute the distance of the vehicle from the next upcoming traffic light, in this we run inference only when close to the traffic light. Based on the detected light, the next waypoint is published. 