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
