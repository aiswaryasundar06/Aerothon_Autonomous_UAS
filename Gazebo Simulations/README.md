GAZEBO SIMULATIONS

Simulations were conducted on Ubuntu 22.04 using ROS 2 Humble and Gazebo Harmonic.A custom SDF arena was created and the PX4 X500 multirotor model was used as the base platform. It was modified by integrating two cameras and four LiDAR sensors to support perception and navigation requirements. 

The system was implemented using three main ROS 2 nodes: 
1) FSM Node: Manages high-level mission logic and state transitions for autonomous navigation. 
2) Perception Node: Processes camera inputs and publishes relevant perception data to ROS 2 topics. 
3) Pilot Node: Uses MAVSDK to bridge ROS 2 with PX4, enabling communication between ROS-based control logic and the PX4 flight stack for executing movement commands and trajectory tracking. 
