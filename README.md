# FiveAges S2 Description

立柱固定底座人形 S2 的 URDF / ros2_control 描述包。模型在 **`xacro/`**；立柱 / 躯干 / 头部在 [`fiveages_w2_common_description`](../fiveages_w2_common_description/README.md)；双臂与末端复用 [`m6_ccs_description`](../../manipulator/Tianji/m6_ccs_description/README.md)（`M6_CCS` 臂、`eefs.xacro`、共享 ros2_control 宏）。

```bash
cd ~/ros2_ws && colcon build --packages-up-to fiveages_s2_description --symlink-install
source install/setup.bash
```

## 1. 可视化与 Launch

| 场景 | 入口 | 示例 |
|------|------|------|
| 全身 RViz | `humanoid.launch.py` | `… robot:=fiveages_s2 collider:=simple` |
| 带末端 | 同上 + `type:=<key>` | `… type:=erg32`（键名见 [m6_ccs §2](../../manipulator/Tianji/m6_ccs_description/README.md#2-末端执行器)） |
| 左右不同末端 | `left_type` / `right_type` | `left_type:=rg75 right_type:=linkerhand_o7` |
| 部件调试 | `component.launch.py` | `… robot:=fiveages_s2`（双臂）；立柱 / 躯干 / 头见 [w2_common](../fiveages_w2_common_description/README.md) |
| 平台子款 | 各 launch + `variant:=<key>` | `variant:=v0` 固定立柱；见 [§2](#2-平台子款variant) |
| 全身 WBC（mock） | `ocs2_arm_controller full_body.launch.py` | 默认 `hardware:=mock_components` |
| 分体控制（mock） | `split_body.launch.py` | 腰 / 头 / 臂独立控制器 |
| 仅腰体 | `basic_joint_controller demo.launch.py` | `robot:=fiveages_s2` |
| Isaac | 上述 launch + `hardware:=isaac` | Topic 桥接仿真 |

```bash
ros2 launch robot_common_launch humanoid.launch.py robot:=fiveages_s2 collider:=simple type:=erg32
ros2 launch robot_common_launch humanoid.launch.py robot:=fiveages_s2 collider:=simple variant:=v0
ros2 launch ocs2_arm_controller full_body.launch.py robot:=fiveages_s2 type:=erg32
ros2 launch ocs2_arm_controller full_body.launch.py robot:=fiveages_s2 variant:=v0
ros2 launch ocs2_arm_controller split_body.launch.py robot:=fiveages_s2 type:=erg32
```

## 2. 平台子款（variant）

Launch 参数 `variant:=<key>`（或 `robot.local.yaml` 中 `platform.variant`）选择立柱与控制配置子款。与 W2 共用同一参数名，但 **S2 特有** 的键如下：

| variant | 立柱 URDF | 腰关节（WBC / split body） | OCS2 / 控制器 |
|---------|-----------|----------------------------|---------------|
| （默认，空） | `lift_joint` 棱柱，可升降 | `lift_joint` [m] + `body_joint1~2` [rad]；`waist_lifting_enabled: true` | `config/ocs2/fixed_base.info`（19 DOF） |
| `v0` | `lift_joint` **fixed**，高度锁定 | 仅 `body_joint1~2`；禁用腰升降话题 | 叠加 `config/ros2_control/v0.yaml` → `fixed_base_v0.info`（18 DOF） |

**说明：**

- `v0` 适用于无升降机构的固定立柱：URDF 将 `lift_joint` 改为 fixed，ros2_control 不暴露其命令接口，WBC / `body_joint_controller` 关节列表与 home 位姿均去掉升降维。
- S2 整机头部固定使用 `stand` 关节限位（偏航 ±1.3963 rad 等），**不**随 launch `variant` 改变；部件预览见 [w2_common §2.3](../fiveages_w2_common_description/README.md#23-head)。
- `variant:=sinopec` 等 OEM 外观键会传入躯干 `torso.xacro` 与 M6 臂 `skin` 子款（规则同 W2），不影响立柱升降逻辑。

```bash
# 固定立柱（无升降）
ros2 launch ocs2_arm_controller full_body.launch.py robot:=fiveages_s2 variant:=v0
ros2 launch ocs2_arm_controller full_body.launch.py robot:=fiveages_s2 variant:=v0 type:=erg32
```

```yaml
# robot.local.yaml 持久化
platform:
  variant: v0
```

## 3. 末端执行器

`type` / `left_type` / `right_type` 与 m6_ccs **完全相同**（解析：`m6_ccs_description/xacro/eef_type_resolver.xacro`）。

真机驱动方式、支持矩阵、CAN 限制等详见 **[m6_ccs README §2](../../manipulator/Tianji/m6_ccs_description/README.md#2-末端执行器)**，此处不重复。

S2 特有：`hardware:=real` 时臂走 Marvin；外接 `usb` / `usb_can` 末端逻辑与 m6_ccs 共用 `external_ee_systems`。立柱驱动尚未接入（`stand_system` 为 mock）。

## 4. 平台与模型

| 层级 | 包 / xacro | 说明 |
|------|------------|------|
| 立柱 | `fiveages_w2_common` `stand.xacro` | `lift_joint` [m] + `body_joint1~2` [rad]；子款见 [§2](#2-平台子款variant) |
| 躯干 / 头 | `fiveages_w2_common` `torso.xacro` / `head.xacro` | 头固定 `stand` 限位；躯干可接 OEM `variant` |
| 双臂 | `m6_ccs` `M6_CCS` | `skin:=fiveages_w2`；碰撞与 m6 共用 `link*.glb` |
| 末端 | `m6_ccs` `eefs.xacro` | 与 m6_ccs 一致 |

OCS2 全身规划：`config/ocs2/fixed_base.info`（`base_frame: base_link`）。分体臂 MPC 使用 `m6_ccs` 的 `task.info`（`ocs2_arm_controller.robot_name: m6_ccs`）。

立柱网格：将 SolidWorks STL 放入 `fiveages_w2_common_description/meshes/stand/`（见该包 `meshes/stand/README.md`）。

## 5. ros2_control 架构

真机拆为 **两个 system**；仿真合并为一个：

| 模式 | System | 硬件插件 |
|------|--------|----------|
| 真机 | `M6_CCS_System` | `marvin_ros2_control`（双臂 + 夹爪/手） |
| 真机 | `stand_system` | mock（`lift_joint`、`body_joint1~2`、`head_joint1~2`；驱动待接入） |
| 真机 | 外接末端 *\_system | `modbus_ros2_control` / `can_ros2_control`（按 drive type） |
| 仿真 | `fiveages_s2_system` | mock / gz / isaac（[`robot_common_launch` simulation_hardware](../../common/robot_common_launch/xacro/ros2_control/simulation_hardware.xacro)） |

手臂侧宏均来自 [m6_ccs §5](../../manipulator/Tianji/m6_ccs_description/README.md#5-ros2_control-共享宏)：`dual_arm_ee_block`、`marvin_hardware_block`、`external_ee_systems` 等。本包 `xacro/ros2_control/robot.xacro` 仅为薄封装。

控制器配置：`config/ros2_control/common.yaml`（`ocs2_wbc_controller` 全身、`ocs2_arm_controller` 双臂、`body_joint_controller` / `head_joint_controller` 等）。`variant:=v0` 时自动叠加 `config/ros2_control/v0.yaml`，详见 [§2](#2-平台子款variant)。末端控制器由 launch 时 `control_compose` 合成，无本包 per-type yaml。

## 6. 真机部署

立柱驱动尚未集成；臂与末端部署方式与 W2 相同：

```bash
colcon build --packages-up-to marvin_ros2_control modbus_ros2_control can_ros2_control --symlink-install
source install/setup.bash

# 仅腰体（mock 立柱）
ros2 launch basic_joint_controller demo.launch.py robot:=fiveages_s2

# 全身真机 + 末端（示例）
ros2 launch ocs2_arm_controller full_body.launch.py robot:=fiveages_s2 hardware:=real type:=ag2f90_c
ros2 launch ocs2_arm_controller full_body.launch.py robot:=fiveages_s2 hardware:=real type:=erg32
ros2 launch ocs2_arm_controller split_body.launch.py robot:=fiveages_s2 hardware:=real_usb type:=linkerhand_o7
```

`robot.local.yaml` 片段（规则同 [W2 README §5](../fiveages_w2_description/README.md#5-真机部署)）：

```yaml
defaults:
  end_effectors:
    type: ag2f90_c
hardware:
  device_ip: 192.168.1.190
  arm_ctrl_mode: position
  max_joint_speed: 50
  max_joint_acceleration: 30
  usb_left_port: /dev/ttyUSB0
  usb_right_port: /dev/ttyUSB2
  left_ee_drive_type: marvin
  right_ee_drive_type: marvin
```

### 分体控制常用话题（`split_body`）

```bash
# 头
ros2 topic pub /head_joint_controller/target_joint_position std_msgs/msg/Float64MultiArray "{data: [0.0, 0.5]}"
# 腰（仅 split_body）：默认 3 维 lift_joint [m] + body_joint1~2 [rad]；variant:=v0 时为 2 维
ros2 topic pub /body_joint_controller/target_joint_position std_msgs/msg/Float64MultiArray \
  "{data: [0.045, 0.0, 0.0]}"
# 腰升降 / 转向（默认款；v0 无升降，仅转向有效）
ros2 topic pub /body_joint_controller/waist_lifting_command std_msgs/msg/Float64 "data: 0.5"
ros2 topic pub /body_joint_controller/waist_turning_command std_msgs/msg/Float64 "data: -0.3"
# 双臂（14 维）
ros2 topic pub /ocs2_arm_controller/target_joint_position std_msgs/msg/Float64MultiArray \
  "{data: [0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0]}"
```

## 7. 导航

S2 为固定底座立柱，**不适用**轮式导航（Nav2 / Linkhou）。轮式 W2 导航见 [fiveages_w2_description README §6](../fiveages_w2_description/README.md#6-导航)。
