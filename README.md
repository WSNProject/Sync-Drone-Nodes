Code Application Description:


 Our main application was to implement a wireless sensor network aidded with unmanned aerial vechile ( In our case it is a drone )

Our WSN consisted of 3 types of nodes : relay node, sensor node & sync node. The Sensor & Relay node were spread in the AoI, the main functionality of these nodes was to monitor and collect the surrounding data ( such as temperature, pressure etc..). They mainly consisted of a waspmote microcontroller,xbee Pro Radio Module and for the sensor node there were additionally sensor board and sensors. 
Furthermore, The most important node in the WSN was the sync node. In our application we only had one sync node. It mainly consisted of waspmote microcontroller,xbee Pro Radio Module, SD card, APC220 High Radio frequency transceivers. The Sync node was positioned in a strategic location for the drone communication. The Sync node was programmed to collect all the data gathered from the other sensor nodes, store this data in the SD card and wait for the Drone(UAV) to arrive. It has a handshaking process of communication with the drone, it keeps on sending a packet with a password which is only recogonized by the node on the drone. Once the drone is in the range of communication of the sync node, it receives the password packet and intiate the handshaking process. If the handshaking process proceeded successfully the data is transferred to the drone node using the  APC220 High Radio frequency transceivers. Thereafter,  it is stored in a SD card. The drone node consist only of waspmote microcontroller, SD card, APC220 High Radio frequency transceivers. 

Finally Once one receive the data from the drone,  one can use a matlab code to parse the packets received and present the data in a userfriendly way rather than a frame 




Folders Description:

The Artichure design: this folder contains a picture, The picture illustrate how we setup our network





Files Description:

Sync & Drone nodes Code : this file contain the waspmote code for both the sync node & the drone node, in addition it also contains the matlab parsing code at the end.
These code need to be used separatly, so each code need to be compiled separately make sure to follow up on the comments inside the code inorder to be able to distinguish the nodes
