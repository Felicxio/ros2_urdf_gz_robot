# ROS 2 Mobile Robot with Robotic Arm and Gazebo Simulation

This project implements a custom mobile robot simulation using **ROS 2**, **URDF/Xacro**, **Gazebo Sim**, **RViz2** and **ros_gz_bridge**.

The robot consists of a differential-drive mobile base, a simple two-joint robotic arm, and an onboard camera. The simulation runs inside a custom Gazebo world with warehouse-like objects, while RViz2 is used to visualize the robot model, TF tree and sensor data.

---

## Demo


[my_robot.webm](https://github.com/user-attachments/assets/02f7fe84-7bfd-455d-bf6a-0656619b2297)


---

## Project Overview

The goal of this project is to practice the full workflow of building and simulating a robot in ROS 2:

- Creating a robot model with URDF and Xacro
- Building a modular robot description
- Simulating the robot in Gazebo Sim
- Connecting Gazebo topics to ROS 2 using `ros_gz_bridge`
- Visualizing the robot in RViz2
- Controlling a differential-drive mobile base
- Controlling revolute joints of a robotic arm
- Publishing camera image and camera info topics

This project is part of my learning path in robotics, ROS 2 and robot simulation.

---

## Features

- Differential-drive mobile robot
- Two-wheel mobile base with caster wheel
- Two-joint robotic arm mounted on the robot base
- Camera sensor integration
- Gazebo Sim environment
- RViz2 visualization
- ROS 2 and Gazebo topic bridge
- Joint position control for the robotic arm
- Joint state publishing
- TF visualization

---

## Technologies Used

- ROS 2 Jazzy
- Gazebo Sim
- RViz2
- URDF
- Xacro
- ros_gz_sim
- ros_gz_bridge
- robot_state_publisher
- CMake / ament_cmake
- Linux / Ubuntu

---

## Repository Structure

```bash
ros2_urdf_gz_robot/
├── src/
│   ├── my_robot_bringup/
│   │   ├── config/
│   │   │   └── gazebo_bridge.yaml
│   │   ├── launch/
│   │   │   └── my_robot_gazebo.launch.xml
│   │   ├── worlds/
│   │   │   └── test_world.sdf
│   │   ├── CMakeLists.txt
│   │   └── package.xml
│   │
│   └── my_robot_description/
│       ├── launch/
│       ├── rviz/
│       ├── urdf/
│       │   ├── my_robot.urdf.xacro
│       │   ├── mobile_base.xacro
│       │   ├── mobile_base_gazebo.xacro
│       │   ├── arm.xacro
│       │   ├── arm_gazebo.xacro
│       │   ├── camera.xacro
│       │   └── common_properties.xacro
│       ├── CMakeLists.txt
│       └── package.xml
└── README.md
```

---

## Robot Description

The robot model is built using modular Xacro files.

The main file is:

```bash
src/my_robot_description/urdf/my_robot.urdf.xacro
```

It includes:

```xml
<xacro:include filename="common_properties.xacro" />
<xacro:include filename="mobile_base.xacro" />
<xacro:include filename="mobile_base_gazebo.xacro" />
<xacro:include filename="arm.xacro" />
<xacro:include filename="arm_gazebo.xacro" />
<xacro:include filename="camera.xacro" />
```

The robotic arm is attached to the mobile base through a fixed joint:

```xml
<joint name="mobile_base_arm_joint" type="fixed">
    <parent link="base_link"/>
    <child link="arm_base_link"/>
</joint>
```

---

## Robotic Arm

The robotic arm has three main links:

- `arm_base_link`
- `forearm_link`
- `hand_link`

And two revolute joints:

- `arm_base_forearm_joint`
- `forearm_hand_joint`

The joints are controlled in Gazebo using the `JointPositionController` plugin.

Example ROS 2 commands:

```bash
ros2 topic pub -1 /joint0/cmd_pos std_msgs/msg/Float64 "{data: 0.5}"
```

```bash
ros2 topic pub -1 /joint1/cmd_pos std_msgs/msg/Float64 "{data: 0.5}"
```

---

## Gazebo Bridge

The bridge configuration is defined in:

```bash
src/my_robot_bringup/config/gazebo_bridge.yaml
```

Main bridged topics:

| ROS 2 Topic | Gazebo Topic | Direction | Purpose |
|---|---|---|---|
| `/clock` | `/clock` | Gazebo to ROS | Simulation time |
| `/joint_states` | Gazebo joint state topic | Gazebo to ROS | Robot joint states |
| `/tf` | Gazebo TF topic | Gazebo to ROS | Robot transforms |
| `/cmd_vel` | Gazebo velocity command topic | ROS to Gazebo | Mobile base control |
| `/camera/image_raw` | Gazebo camera image topic | Gazebo to ROS | Camera image |
| `/camera/camera_info` | Gazebo camera info topic | Gazebo to ROS | Camera calibration/info |
| `/joint0/cmd_pos` | Gazebo arm joint command topic | ROS to Gazebo | First arm joint control |
| `/joint1/cmd_pos` | Gazebo arm joint command topic | ROS to Gazebo | Second arm joint control |

---

## How to Run

Clone the repository inside a ROS 2 workspace:

```bash
mkdir -p ~/l2_ros2_ws/src
cd ~/l2_ros2_ws/src
git clone https://github.com/Felicxio/ros2_urdf_gz_robot.git
```

Go back to the workspace root:

```bash
cd ~/l2_ros2_ws
```

Build the packages:

```bash
colcon build
```

Source the workspace:

```bash
source install/setup.bash
```

Launch the simulation:

```bash
ros2 launch my_robot_bringup my_robot_gazebo.launch.xml
```

This launch file starts:

- `robot_state_publisher`
- Gazebo Sim
- robot spawning from `robot_description`
- `ros_gz_bridge`
- RViz2

---

## Control the Mobile Base

Publish velocity commands to move the robot:

```bash
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist "{
  linear: {x: 0.3, y: 0.0, z: 0.0},
  angular: {x: 0.0, y: 0.0, z: 0.0}
}"
```

Rotate the robot:

```bash
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist "{
  linear: {x: 0.0, y: 0.0, z: 0.0},
  angular: {x: 0.0, y: 0.0, z: 0.5}
}"
```

Stop the robot:

```bash
ros2 topic pub -1 /cmd_vel geometry_msgs/msg/Twist "{
  linear: {x: 0.0, y: 0.0, z: 0.0},
  angular: {x: 0.0, y: 0.0, z: 0.0}
}"
```

---

## Control the Robotic Arm

Move the first arm joint:

```bash
ros2 topic pub -1 /joint0/cmd_pos std_msgs/msg/Float64 "{data: 0.5}"
```

Move the second arm joint:

```bash
ros2 topic pub -1 /joint1/cmd_pos std_msgs/msg/Float64 "{data: 0.5}"
```

The joint values are position commands in radians.

---

## Useful Debug Commands

Check ROS 2 topics:

```bash
ros2 topic list
```

Check the arm command topic:

```bash
ros2 topic info /joint0/cmd_pos -v
```

Check joint states:

```bash
ros2 topic echo /joint_states
```

Check Gazebo topics:

```bash
gz topic -l
```

Filter Gazebo joint command topics:

```bash
gz topic -l | grep cmd_pos
```

Filter Gazebo joint state topics:

```bash
gz topic -l | grep joint
```

View the TF tree:

```bash
ros2 run tf2_tools view_frames
```

---

## Current Status

Implemented:

- Mobile robot URDF/Xacro model
- Gazebo simulation
- RViz2 visualization
- Differential drive plugin
- Robotic arm model
- Joint position control for the arm
- Joint state publisher
- Camera topic bridge
- Custom Gazebo world

---

## Next Steps

Possible improvements:

- Add a gripper to the robotic arm
- Add keyboard or joystick teleoperation
- Add ROS 2 controllers
- Add MoveIt2 for motion planning
- Improve camera positioning and perception pipeline
- Add object detection or computer vision
- Add autonomous navigation
- Improve the Gazebo world and interaction with objects
- Add a launch argument to switch between RViz-only and Gazebo simulation modes

---

## Lessons Learned

During this project, I practiced important ROS 2 and robotics simulation concepts:

- How to organize a ROS 2 workspace with multiple packages
- How to create modular URDF/Xacro files
- How to connect links and joints correctly
- How TF depends on `robot_state_publisher` and joint states
- How to debug missing transforms in RViz2
- How to use Gazebo plugins for differential drive and joint control
- How to bridge Gazebo Sim topics into ROS 2
- How to test robot commands from the terminal

---

## Author

Developed by João Victor Assunção Pereira.

This project is part of my robotics and ROS 2 learning journey, combining my background in Mechatronics Engineering with practical simulation, robot modeling and control.
