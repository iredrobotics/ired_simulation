# ired_simulation

ROS 2 **Jazzy** + **Gazebo Harmonic (gz sim 8)** simulation for the `ired` differential-drive
robot. Description, config, and launch only — motion and sensors are gz-native (no `ros2_control`).

## Prerequisites

- ROS 2 Jazzy and the `ros_gz` stack (`ros_gz_sim`, `ros_gz_bridge`), `laser_filters`, and the
  gz Harmonic plugins.
- `nav2_minimal_tb3_sim` — supplies the `turtlebot3_world` model loaded by the default world.

These are the only packages required; all are expected to be already installed.

## Build

Build from the **workspace root** (`~/ired_ws`), then source the overlay:

```bash
cd ~/ired_ws
colcon build --packages-select ired_simulation
source install/setup.bash
```

Descriptions/launch/config are read from the **install** share, so rebuild after editing them.

## Run

Full simulation (gz sim + robot + bridge + laser filter):

```bash
ros2 launch ired_simulation gazebo.launch.xml
```

Model in RViz only, no Gazebo:

```bash
ros2 launch ired_simulation display.launch.xml
```

### Launch arguments (`gazebo.launch.xml`)

| Arg | Default | Effect |
| --- | --- | --- |
| `world` | `turtlebot3_world` | World from `worlds/<world>.world`. |
| `gui` | `true` | `false` runs the server only (headless). |
| `rviz` | `false` | `true` also opens RViz. |
| `use_sim_time` | `true` | Use `/clock` from gz sim. |
| `use_laser_filters` | `true` | `false` disables the `/scan → /scan_filtered` chain. |
| `x` `y` `z` `roll` `pitch` `yaw` | `-2.0 -0.5 0.1 0 0 0` | Robot spawn pose. |

List all args without launching:

```bash
ros2 launch ired_simulation gazebo.launch.xml --show-args
```

## Topics

Bridged between gz and ROS via `config/ros_gz_bridge.yaml`:

| Topic | ROS type | Direction | Notes |
| --- | --- | --- | --- |
| `/clock` | `rosgraph_msgs/msg/Clock` | gz → ROS | Sim time. |
| `/cmd_vel` | `geometry_msgs/msg/TwistStamped` | ROS → gz | Drive command (mapped to unstamped `gz.msgs.Twist`). |
| `/odom` | `nav_msgs/msg/Odometry` | gz → ROS | Wheel odometry from DiffDrive. |
| `/tf` | `tf2_msgs/msg/TFMessage` | gz → ROS | Includes `odom → base_footprint`. |
| `/joint_states` | `sensor_msgs/msg/JointState` | gz → ROS | From the JointState system. |
| `/scan` | `sensor_msgs/msg/LaserScan` | gz → ROS | Lidar on `base_scan`. |
| `/ired/imu/data` | `sensor_msgs/msg/Imu` | gz → ROS | IMU on `base_link`. |
| `/model/ired/pose` | `tf2_msgs/msg/TFMessage` | gz → ROS | Ground-truth robot pose in the world. |

Produced ROS-side (not bridged):

| Topic | ROS type | Source |
| --- | --- | --- |
| `/scan_filtered` | `sensor_msgs/msg/LaserScan` | `laser_filters` (when `use_laser_filters:=true`). |
| `/robot_description` | `std_msgs/msg/String` | `robot_state_publisher` (latched). |

Inspect a running sim:

```bash
ros2 topic list
ros2 topic echo /odom
```
