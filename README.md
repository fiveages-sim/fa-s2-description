# FiveAges S2 Description

This package contains the description files for FiveAges S2 stand humanoid (fixed-base column waist + shared torso/head + M6 dual arms).

## 1. Build

```bash
cd ~/ros2_ws
colcon build --packages-up-to fiveages_s2_description --symlink-install
```

## 2. Visualize the robot

### 2.1 Full Robot

* FiveAges S2 (Basic visualization)
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch humanoid.launch.py robot:=fiveages_s2 collider:=simple
  ```

<details>
<summary><strong>More Configs with End Effector</strong></summary>

* With ChangingTek AG2F90 Gripper
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch humanoid.launch.py robot:=fiveages_s2 collider:=simple type:=ag2f90
  ```
* With Jodell RG75 Gripper
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch humanoid.launch.py robot:=fiveages_s2 collider:=simple type:=rg75
  ```

</details>

### 2.2 Component

* Dual arms (default)
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch robot_common_launch component.launch.py robot:=fiveages_s2
  ```

Stand, torso, and head component visualization live in `fiveages_w2_common_description`. See [fiveages_w2_common_description/README.md](../fiveages_w2_common_description/README.md) (`type:=stand`, `type:=torso`, `type:=head`).

## 3. ROS2 Control Demos

### 3.1 Mock Component

* Full Body Control
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller full_body.launch.py robot:=fiveages_s2
  ```
* Split Body Control (arms via OCS2 + body/head via `basic_joint_controller`)
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller split_body.launch.py robot:=fiveages_s2
  ```
* Only Body
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch basic_joint_controller demo.launch.py robot:=fiveages_s2
  ```

<details>
<summary><strong>More Full Body Configs</strong></summary>

* With ChangingTek AG2F90 Gripper
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller full_body.launch.py type:=ag2f90 robot:=fiveages_s2
  ```
* With Jodell RG75 Gripper
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller full_body.launch.py type:=rg75 robot:=fiveages_s2
  ```

</details>

<details>
<summary><strong>More Split Body Configs</strong></summary>

* With ChangingTek AG2F90 Gripper
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller split_body.launch.py type:=ag2f90 robot:=fiveages_s2
  ```
* With Jodell RG75 Gripper
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller split_body.launch.py type:=rg75 robot:=fiveages_s2
  ```

</details>

### 3.2 Isaac Sim

* Empty Hand (Full Body Control)
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller full_body.launch.py robot:=fiveages_s2 hardware:=isaac
  ```
* With Jodell RG75 Gripper (Split Body)
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller split_body.launch.py type:=rg75 robot:=fiveages_s2 hardware:=isaac
  ```
* With ChangingTek AG2F90 Gripper (Split Body)
  ```bash
  source ~/ros2_ws/install/setup.bash
  ros2 launch ocs2_arm_controller split_body.launch.py type:=ag2f90 robot:=fiveages_s2 hardware:=isaac
  ```

### 3.3 Joint Control Commands

* Set Head Angles
  ```bash
  ros2 topic pub /head_joint_controller/target_joint_position std_msgs/msg/Float64MultiArray "{data: [0.0, 0.5]}"  # 设置目标关节角度（弧度）
  ```
  ```bash
  ros2 topic pub /head_joint_controller/target_joint_position std_msgs/msg/Float64MultiArray "{data: [0.0, 0.0]}"  # 设置目标关节角度（弧度）
  ```
* Set Body Angles (only available on split control)
  ```bash
  ros2 topic pub /body_joint_controller/target_joint_position std_msgs/msg/Float64MultiArray "{data: [0.0, 0.0, 0.0]}"  # body_joint1 [m], body_joint2~3 [rad]
  ```
  ```bash
  ros2 topic pub /body_joint_controller/target_joint_position std_msgs/msg/Float64MultiArray "{data: [0.045, 0.0, 0.785]}"  # 设置目标关节角度（弧度）
  ```
* Waist Lift / Turn (split control, `waist_lifting_type: single_joint`)
  ```bash
  # 升降：body_joint1 单轴 [m]；转向：body_joint2 [rad]
  ros2 topic pub /body_joint_controller/waist_lifting_command std_msgs/msg/Float64 "data: 0.5"   # 上升，发 0.0 停止
  ros2 topic pub /body_joint_controller/waist_turning_command std_msgs/msg/Float64 "data: -0.3"  # 左转，发 0.0 停止
  ```
  ```bash
  ros2 topic pub --once /body_joint_controller/waist_lifting std_msgs/msg/Float64 "data: -0.05"  # 相对上升 5 cm（关节增大=下降，取负）
  ```
* Set Arm Angles
  ```bash
  ros2 topic pub /ocs2_arm_controller/target_joint_position std_msgs/msg/Float64MultiArray "{data: [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0,0.0,0.0,0.0]}"  # 设置目标关节角度（弧度）
  ```

## 4. Real Robot Deploy

Stand hardware driver is not integrated yet (mock only). Arm real-robot deploy follows the same Marvin stack as W2 when enabled in `xacro/ros2_control/robot.xacro`.

## 5. Navigation

S2 is a fixed-base stand robot; mobile navigation (Nav2 / Linkhou chassis) is not applicable. For W2 wheel-base navigation, see [fiveages_w2_description/README.md](../fiveages_w2_description/README.md).

Stand meshes: copy SolidWorks STL into `fiveages_w2_common_description/meshes/stand/` (see that package `meshes/stand/README.md`). OCS2 fixed-base config: `config/ocs2/fixed_base.info` (`base_frame: base_link`).

Split-body arm MPC uses `m6_ccs_description/config/ocs2/task.info` (`ocs2_arm_controller.robot_name: m6_ccs` in `common.yaml`); no local `task_arm.info` is required.
