# KUKA LWR 4+ 

**Works on ROS Melodic**

## Overview

The main packages are:
- [__lwr_description__](lwr_description): a package that defines the model of the robot (ToDo: name it __lwr_model__)
- [__lwr_hw__](lwr_hw): a package that contains the LWR 4+ definition within the ros control framework, and also final interfaces using Kuka FRI, Stanford FRI Library or a Gazebo plugin. Read adding an interface below if you wish to add a different non-existing interface. 
- [__lwr_controllers__](lwr_controllers): a package that implement a set of useful controllers (ToDo: perhaps moving this to a forked version of `ros_controllers` would be ok, but some controllers are specific for the a 7-dof arm).
- [__single_lwr_example__](single_lwr_example): a cofiguration-based meta-package that shows how to use the `kuka_lwr` packages.
	- [__single_lwr_robot__](single_lwr_example/single_lwr_robot): the package where you define your robot using the LWR 4+ arm.
	- [__single_lwr_moveit__](single_lwr_example/single_lwr_moveit): the moveit configuration for your `single_lwr_robot` description.
	- [__single_lwr_launch__](single_lwr_example/single_lwr_launch): a launch interface to load different components and configuration of your setup be it real, simulation, moveit, visualization, etc.

For an example using two LWR 4+ arms and two Pisa/IIT SoftHands, see the [Vito robot](https://github.com/CentroEPiaggio/vito-robot).

## Adding more interfaces/platforms

The package [lwr_hw](lwr_hw) contains the abstraction that allows to make the most of the new ros control framework. To create an instance of the arm you need to call the function [`void LWRHW::create(std::string name, std::string urdf_string)`](lwr_hw/include/lwr_hw/lwr_hw.h#L40), where `name` MUST match the name you give to the xacro instance in the URDF and the `urdf_string` is any robot description containing one single instance of the lwr called as `name` (note that, several lwr can exist in `urdf_string` if they are called differently).

The abstraction is enforced with [three pure virtual functions](lwr_hw/include/lwr_hw/lwr_hw.h#L61-L64):
``` c++
  virtual bool init() = 0;
  virtual void read(ros::Time time, ros::Duration period) = 0;
  virtual void write(ros::Time time, ros::Duration period) = 0;
```

Adding an interface boils down to inherit from the [LWRHW class](lwr_hw/include/lwr_hw/lwr_hw.h#L33), implement these three function according to your final platform, and creating either a node or a plugin that uses your new class. This way you can use all your planning and controllers setup in any final real or simulated robot.

Examples of final interface class implementations are found for the [Kuka FRI](lwr_hw/include/lwr_hw/lwr_hw_fri.hpp), [Stanford FRI library](lwr_hw/include/lwr_hw/lwr_hw_fril.hpp) and a [Gazebo simulation](lwr_hw/include/lwr_hw/lwr_hw_gazebo.hpp). The corresponding nodes and plugin are found [here](lwr_hw/src/lwr_hw_fri_node.cpp), [here](lwr_hw/src/lwr_hw_fril_node.cpp), and [here](lwr_hw/src/lwr_hw_gazebo_plugin.cpp).

## How to run a real LWR 4+

1. Load the script [`lwr_hw/krl/ros_control.src`](lwr_hw/krl/ros_control.src) and the corresponding [`.dat`](lwr_hw/krl/ros_control.src) on the robot. 
2. Place the robot in a position where joints 1 and 3 are as bent as possible (at least 45 degrees) to avoid the "__FRI interpolation error__" message. A good way to check this is to go into gravity compensation, and see if the robot enters succesfully in that mode with the configured tool.
3. Set the robot in __Position__ control.
4. Start the script with the grey and green buttons; the scripts stops at a point and should be started again by releasing and pressing again the green button. You can also use the script in semi-automatic mode. 
5. Start the ROS node, controllers will not start until the handshake is done.
6. From this point on, you can manage/start/stop/run/switch controllers from ROS, and depending on which interface they use, the switch in the KRC unit is done automatically. Everytime there is a switch of interface, it might take a while to get the controllers running again. If the controllers use the same interface, the switch is done only in ROS, which is faster.

### Additional information to use the Stanford FRI Library (not fully tested)

You need to provide your user name with real time priority and memlock limits higher than the default ones. You can do it permanently like this:

1. `sudo nano /etc/security/limits.conf` and add these lines: 
```
YOUR_USERNAME hard rtprio 95
YOUR_USERNAME soft rtprio 95
YOUR_USERNAME hard memlock unlimited
YOUR_USERNAME soft memlock unlimited
```
2. `sudo  nano /etc/pam.d/common-session` and add `session required pam_limits.so`
3. Reboot, open a terminal, and check that `ulimit -r -l` gives you the values set above.
4. You need to edit manually the [lwr_hw.launch](lwr_hw/launch/lwr_hw.launch)
5. Load the KRL script that is downloaded with the library to the Robot, and follow the instructions from the [original link](http://cs.stanford.edu/people/tkr/fri/html/) to set up network and other requirements properly.
