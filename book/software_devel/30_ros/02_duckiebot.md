# ROS on the Duckiebot {#ros-duckiebot status=draft}

## ROS Version

The version of ROS we are running on the duckiebots is ROS Kinetic. The current long term service version is ROS Melodic however it is quite new and not as well documented as Kinetic which will be supported until April 2021. You can install Melodic on your personal laptop and will be able to communicate with the duckiebot but may have issues building the duckiebot software on your own laptop.

## ROS Installation

See the [official guide](http://wiki.ros.org/ROS/Installation) for installing ROS.


## ROS Master

The ROS master is a process which manages the connections between different nodes. It is typically run using the `roscore` command. We launch a container specifically for the ROS master with:

	laptop $ docker -H ![hostname].local run -dit --privileged --name roscore --net host --restart unless-stopped duckietown/rpi-ros-kinetic-roscore:master18


Let's try to communicate with ROS master on our laptops. First we need to tell our computer where ROS master is. we do this by setting the environment variable `ROS_MASTER_URI` with the command

	laptop $ export ROS_MASTER_URI=http://![hostname].local:11311/

Keep in mind this variable setting will not persist if you close the terminal. One option is to put that line in your `.bashrc` or `.zshrc` but then you wont be able to connect to any ROS master on your machine unless you reset it.

