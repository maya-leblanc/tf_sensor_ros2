# Differential-Drive Mobile Robot — ROS 2 Simulation & LiDAR Integration

A simulated differential-drive mobile robot built for **ROS 2 Jazzy** and **Gazebo Harmonic**.
The robot is modeled in URDF/Xacro, driven through the `ros_gz` bridge, and equipped with a
360° LiDAR whose scans are correctly localized to the robot via the TF transform tree.
Controlled in real time with keyboard teleoperation.

![Robot in Gazebo](docs/gazebo.png)
<!-- replace with your own screenshot: robot + obstacles in Gazebo -->

## Overview

This project implements the full simulation stack for a two-wheeled mobile robot:
mechanical model → physics simulation → sensor integration → teleoperation. It was built
as a from-scratch exercise in ROS 2 middleware, the modern Gazebo (`gz-sim`) plugin system,
and coordinate-frame (TF) management — the foundations of any mobile-robot software stack.

**Stack:** ROS 2 Jazzy · Gazebo Harmonic · URDF/Xacro · `ros_gz_bridge` · RViz2 · Ubuntu 24.04

## Features

- **Parametric robot model** — chassis, two drive wheels, caster, and LiDAR mast defined in
  Xacro with reusable macros for links and inertia tensors (box, cylinder, sphere).
- **Differential-drive control** — the `gz-sim-diff-drive-system` plugin converts velocity
  commands into per-wheel motion and publishes wheel odometry.
- **LiDAR perception** — a 360° `gpu_lidar` sensor publishes range scans, frame-stamped to
  the `lidar_link` so they resolve correctly in the TF tree.
- **TF transform tree** — `robot_state_publisher` and the drive plugin jointly build
  `odom → base_footprint → base_link → {wheels, lidar_link}`, letting scans be placed in
  the world frame as the robot moves.
- **ROS ↔ Gazebo bridge** — `/cmd_vel`, `/odom`, `/tf`, `/joint_states`, `/scan`, and `/clock`
  bridged between ROS 2 and Gazebo.
- **Keyboard teleoperation** — drive the robot live via `teleop_twist_keyboard`.

## Requirements

- Ubuntu 24.04
- ROS 2 Jazzy
- Gazebo Harmonic

Install dependencies:

```bash
sudo apt install ros-jazzy-ros-gz ros-jazzy-ros-gz-bridge \
  ros-jazzy-xacro ros-jazzy-robot-state-publisher \
  ros-jazzy-teleop-twist-keyboard ros-jazzy-rviz2
```

## Build & run

```bash
mkdir -p ~/ros2_ws/src && cd ~/ros2_ws/src
git clone <your-repo-url> my_robot
cd ~/ros2_ws
colcon build --packages-select my_robot
source install/setup.bash
ros2 launch my_robot sim.launch.py
```

Gazebo opens with the robot and obstacles. In a second terminal, drive it:

```bash
source /opt/ros/jazzy/setup.bash
source ~/ros2_ws/install/setup.bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

Use `i / j / k / l / ,` to move and `q / z` to change speed (keep that terminal focused).

## Visualizing the LiDAR

Launch RViz2, set **Fixed Frame** to `odom`, and add **RobotModel** (`/robot_description`),
**TF**, and **LaserScan** (`/scan`) displays. The scan points trace the obstacles and follow
the robot as you drive — confirming the sensor is correctly integrated through TF.

![RViz laser scan](docs/rviz.png)
<!-- replace with your own screenshot: RViz showing the laser scan -->

## How it works

**Teleoperation → motion.** `teleop_twist_keyboard` publishes `geometry_msgs/Twist` (linear +
angular velocity) on `/cmd_vel`. The bridge forwards it to Gazebo, where the differential-drive
plugin solves the drive kinematics into left/right wheel speeds — you command a velocity, the
plugin handles the wheels.

**TF transform tree.** `robot_state_publisher` reads the URDF and live wheel angles (from
`/joint_states`) to publish the robot's internal frames; the drive plugin publishes
`odom → base_footprint`. Chained, every part of the robot — and every LiDAR scan — has a known
position in the world frame.

**LiDAR via TF.** Each scan is stamped with `frame_id = lidar_link`. Because TF knows where
`lidar_link` sits relative to the robot and the world, the scan data is placed correctly in
space as the robot drives — the basis for mapping and navigation.

## Roadmap

- [ ] 2D occupancy mapping with SLAM Toolbox (`/scan` → map)
- [ ] Autonomous navigation with Nav2
- [ ] Hardware deployment on a Raspberry Pi–based chassis

## License

MIT
