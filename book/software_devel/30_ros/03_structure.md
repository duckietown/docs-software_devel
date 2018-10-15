# Code Structure {#ros-structure status=draft}

Software development in ROS takes place in a root directory call a catkin workspace, usually called `catkin_ws`. Inside this directory is a `src` folder which contains all the packages. ROS is built be several packages which each perform a specific role such as defining rosnodes, actions or message definitions. 

See [joy_mapper](https://github.com/duckietown/Software/tree/master18/catkin_ws/src/05-teleop/joy_mapper) as an example of a ROS package. It can be launched independently with

	duckiebot $ roslaunch launch/joy_mapper_test.launch

however we usually launch these nodes as part of a container.

## Catkin Tools

Catkin tools are recommended they let you use `catkin build` rather than `catkin_make` as well as several other commands.

## Creating Nodes

Best practice is to define a class which contains several member functions. You can bind those functions as callback functions when subscribing to topic or call on them when you want to publish something. You should also make use of timers rather than use while loops which block.

For example in joy_mapper the JoyMapper subscribes to the "joy" topic with callback function `cbJoy`.

joy_mapper also uses a timer with callback `cbParamTimer` which is called every second to update parameters.

Notice that the main function simply creates the rosnode, instantiates the JoyMapper class then calls rospy.spin() which keeps the process at that line until it is called to exit.