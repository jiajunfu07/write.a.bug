# W11 · ros2_control（课件）

> 对应阅读: M Ch6 (p.217–242)
> 里程碑: 控制器演示（diff_drive_controller 跑起来）
> 配套笔记: 待写（建议命名 `Chapter-11-ros2_control.md`）

## 学习目标

1. 说清 ros2_control 要解决什么问题（"仿真/真机切换时只换硬件接口"）；
2. 掌握四大件：controller_manager、controller、hardware interface、YAML 描述；
3. 用 `diff_drive_controller` 驱动 W10 的机器人并理解数据流。

## 核心概念

### 1. 为什么需要 ros2_control

没有它：控制器逻辑直接写死对"某块硬件"的读写，换硬件（Gazebo↔真机 CAN 总线）要重写。
有了它：**控制器只面向"硬件接口"编程**；硬件接口负责"把请求落到具体硬件"。

```
        /cmd_vel
            ▼
┌──────────────────────┐
│   controller_manager  │  (一个节点, 负责调度/状态机)
│  ┌──────────────────┐ │
│  │diff_drive_ctrl   │ │  (插件: 把 cmd_vel 算成关节请求)
│  └──────────────────┘ │
├──────────────────────┤
│  hardware_interface   │  (插件: Gazebo 系统 / 真机驱动)
└──────────────────────┘
            ▼
      物理硬件 / 仿真
```

### 2. 四大件

| 组件 | 形态 | 职责 |
|---|---|---|
| controller_manager | 常驻节点 | 统一 update 循环、控制器生命周期管理 |
| controller（插件） | 如 diff_drive_controller、joint_trajectory_controller | 控制算法：cmd→关节指令 |
| hardware_interface（插件） | 如 gz 的 diff drive system、真机 CAN 驱动 | 读写真实/仿真硬件 |
| 描述文件（YAML） | ros__parameters | 声明谁挂谁、参数 |

### 3. 状态机

`unconfigured → configured → active`（还有 error、inactive）；切换用：

```bash
ros2 control list_controllers
ros2 control set_controller_state /diff_drive_controller active
ros2 control list_hardware_interfaces
```

### 4. YAML 最小示例

```yaml
controller_manager:
  ros__parameters:
    update_rate: 100          # Hz
    hardware_plugins:
      - my_robot::MyRobotSystem      # 你实现的硬件接口
    diff_drive_controller:
      type: diff_drive_controller/DiffDriveController
      left_joint: [left_wheel_joint]
      right_joint: [right_wheel_joint]
      wheel_separation: 0.36
      wheel_radius: 0.05
```

（Gazebo Sim 环境下，"hardware interface" 常由 gz 的 diff-drive system 插件承担，ros2_control 负责上层统一接口。）

## 动手步骤

1. 按 M Ch6 Step 1–4 建包、加控制器描述、改 launch（书中有完整模板，照抄后逐行注释）。
2. 启动后 `ros2 control list_controllers`，把 diff_drive_controller 设为 active。
3. `ros2 topic pub -r 10 /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.15}}"` 驱动，观察仿真/轮子转动。
4. `ros2 topic echo /odom`（若启用 odom 发布）验证里程计链路。
5. （进阶）写一个最小 forward controller 插件（M Ch6 p.230–239）。

## 常见坑

- 控制器状态没切到 active → 发了 /cmd_vel 没反应（先 `list_controllers` 看状态）。
- YAML 里 joint 名与 URDF 不一致 → update 循环里报"接口缺失"。
- `update_rate` 设 1000Hz → CPU 飙升，仿真变慢；从 50–100Hz 起步。
- URDF 里需要 `<ros2_control>` 标签声明 hardware interface（Gazebo 场景下格式以 M Ch6 为准）——漏了会"插件找不到硬件"。
- 同一关节被两个控制器同时 claim → 冲突报错；一次只给一个 controller。

## 自测

1. ros2_control 的"可移植性"体现在哪一层？
2. 四大件各自是什么形态（节点/插件/文件）？
3. 控制器的 4 个状态及切换命令？
4. `update_rate` 控制的是什么？
5. 同一个 joint 能被两个 active controller 同时控制吗？为什么？

### 参考答案

1. 硬件接口层：控制器与算法不变，只替换 hardware_interface 插件即可从仿真切真机。
2. controller_manager=节点；controller=插件（算法）；hardware interface=插件（硬件）；描述=ros__parameters YAML。
3. unconfigured/configured/inactive(=configured 未激活)/active/error；`ros2 control set_controller_state`。
4. 整个控制-硬件 update 循环的频率（每个周期里所有 controller 和 hw 各跑一次 read/update/write）。
5. 不能：接口独占，claim 冲突会导致未定义行为，manager 会拒绝/报错。

## 笔记清单（对照书本补写）

- [ ] 画 ros2_control 架构图（cmd → manager → controller → hw → 硬件）
- [ ] 抄写 YAML 模板 + 每行注释
- [ ] 记录 `ros2 control` 三条命令的完整输出
- [ ] 用自己的话写"从 Gazebo 换到真机，要改哪几个文件、不改哪几个"
- [ ] 整理控制器状态机图 + 每个状态的含义
