# Useful Commands {#ros-commands status=draft}
Here are just a few useful commands that can help with debugging


## Nodes, Topics, Services

See all topic names:

	laptop $ rostopic list

Print topic messages to terminal:

	laptop $ rostopic echo ![topic name]

See frequency of messages published on topic:

	laptop $ rostopic hz ![topic name]

See which node is publishing and subscribing to a topic

	laptop $ rosotpic info ![topic name]

## rqt

rqt is a collection of packages and tools for debugging in ROS

See a graph of all nodes and topics:

	laptop $ rqt_graph

Here is an example graph.

<img src="pics/rqt_graph.png" id="rqt_graph"/>

<style>
#rqt_graph {
width: 30em;
}
figure img {
max-width: 100%;
}
</style>


## rviz

rviz is a tool for visualizing data from topics.

	laptop $ rviz

On the left panel you can add visualizations based on message type for topic.