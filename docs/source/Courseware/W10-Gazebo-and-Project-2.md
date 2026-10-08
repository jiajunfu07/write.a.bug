# W10 · Gazebo 仿真 + 项目二（课件）

> 对应阅读: S Ch13；M Ch5 (p.177–214)
> 里程碑: **项目二 · 自定义机器人仿真** + 阶段考 2
> 配套笔记: 待写（建议命名 `Chapter-10-Gazebo-Simulation.md`）

## 学习目标

1. 说清 Gazebo Sim（gz）的 server/client 架构与插件机制；
2. 把自己的 URDF 机器人送进 Gazebo 并用 `cmd_vel` 驱动；
3. 在仿真中接入传感器（LiDAR/相机）并在 RViz 看数据；
4. 按验收清单交付项目二。

## 核心概念

### 1. Gazebo Sim 架构

```
gz sim (server)                     GUI client
┌─────────────────────┐            ┌──────────────┐
│ world + 物理引擎      │  渲染数据   │ 3D 窗口       │
│ system plugins:     │ ─────────▶ │ (可无, 无头运行)│
│  DiffDrive, Sensors,│            └──────────────┘
│  JointStatePublisher│
│ + ROS 2 桥 (内置)     │
└─────────────────────┘
```

- **Jazzy 配套 Gazebo Harmonic**，自带 ROS 2 支持（不用单独装 ros_gz bridge）。
- 与 Gazebo Classic（9）的区别：新的 plugin/SDF 体系、C++20、`gz sim` 命令、system 插件模型。
- 无显示环境自动退化为 server-only（headless 友好）。

### 2. URDF → SDF → 仿真

```bash
sudo apt install ros-jazzy-gz-ros -y        # Gazebo + ROS2 集成
urdf_to_sdf my_robot.urdf > my_robot.sdf    # URDF 只是结构, SDF 才能带 world
gz sim -r my_robot.sdf                      # -r: 随机位置
```

### 3. 差速驱动（URDF 里声明 gz 插件）

```xml
<gazebo>
  <plugin filename="gz-sim-diff-drive-system" name="gz::sim::systems::DiffDrive">
    <left_joint>left_wheel_joint</left_joint>
    <right_joint>right_wheel_joint</right_joint>
    <wheel_separation>0.36</wheel_separation>
    <wheel_radius>0.05</wheel_radius>
  </plugin>
</gazebo>
```

之后 `ros2 topic pub /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.2}, angular: {z: 0.5}}"` 即可驱动。

### 4. 传感器（LiDAR 示例）

URDF 的 `<gazebo>` 里加 `gz-sim-sensors-system` 插件（lidar 类型），输出话题 `/scan`（`sensor_msgs/LaserScan`），RViz 加 LaserScan 显示。

### 5. 世界文件（SDF world）

`<model name="ground_plane"/>`、光照、自定义障碍物 model；`gz sim world.sdf my_robot.sdf` 可同时载入。

## 动手步骤

1. `sudo apt install ros-jazzy-gz-ros ros-jazzy-gazebo-diff-drive-system-plugin -y`（具体包名以 S Ch13 为准）。
2. 给 W08 机器人补 `<gazebo>` 插件（diff drive + 一个 lidar）。
3. `urdf_to_sdf` → `gz sim` 启动 → RViz 显示 LaserScan + RobotModel + TF。
4. `cmd_vel` 驱动：直线 2s、转弯 2s，观察 TF 中 odom→base 变化（若配置了 odom 发布）。
5. 录 3 分钟演示视频（项目二验收）。

## 项目二验收清单

- [ ] W8/W9 的机器人在 Gazebo 中 spawn 成功（截图/视频）
- [ ] `cmd_vel` 话题驱动直线 + 转弯
- [ ] 至少 1 个传感器出数据，RViz 同步显示
- [ ] `/tf` 树完整（map→odom→base 或 base→link）
- [ ] 一个 launch 文件一键起"仿真 + RViz"
- [ ] README：环境、启动、验证命令、已知问题

## 阶段考 2 范围

- TF 树结构题（给描述画图 / 找环）
- URDF：joint 类型、inertial 必要性、xacro 宏
- Gazebo：server/client 职责、插件机制、URDF vs SDF
- 差速运动学：v、ω 与左右轮速的换算（推导一遍）

## 常见坑

- 穿地/爆炸 → 忘 inertial 或 inertial 原点偏了（回到 W08 检查三块 origin）。
- 驱动无反应 → gz 插件没在 URDF 声明、或 joint 名与插件配置对不上、或 `RMW_IMPLEMENTATION` 两侧不一致。
- RViz 看不到 /scan → `use_sim_time` 没设（仿真时间！订阅端要 `parameters=[{'use_sim_time': True}]`）。
- `gz sim` 报 plugin 加载失败 → 包没装（`gz-sim-diff-drive-system-plugin` 等）或版本不匹配。
- 多开仿真 → 话题名打架；用不同 `ROS_DOMAIN_ID` 或私有命名空间隔离。
- 时间戳：仿真里所有节点建议统一 `use_sim_time: true`，否则 tf2 lookup 报时间错。

## 自测

1. Gazebo Sim 的 server 与 client 各负责什么？没有 client 能仿真吗？
2. URDF 与 SDF 的关系？为什么 `urdf_to_sdf`？
3. `use_sim_time` 是什么？不设会有什么症状？
4. 差速驱动：给定 v=0.2 m/s、ω=1.0 rad/s、轮距 L=0.36 m，求左右轮线速度。
5. 仿真里"传感器数据 → 决策"的最小链路是什么（话题级）？

### 参考答案

1. server 跑物理/世界/插件；client 是渲染窗口（可选）。能，无显示时自动 server-only。
2. URDF 只描述机器人结构；SDF 描述"世界 + 机器人 + 插件"，仿真入口是 SDF；urdf_to_sdf 把结构搬进 SDF。
3. "用仿真时钟做消息时间戳"；不设会出现 tf2 时间错、RViz 数据不刷新、时间戳为真实时间等混乱。
4. v_l = v + ωL/2 = 0.2+0.18=0.38；v_r = v − ωL/2 = 0.02 m/s。
5. sensor 插件 → /scan（等）→ 感知/导航节点 → /cmd_vel → diff drive 插件 → 物理运动。

## 笔记清单（对照书本补写）

- [ ] 画 Gazebo Sim 架构图（server/client/plugin/ROS2 桥）
- [ ] 抄写 URDF 中的 gz 插件声明（diff drive + lidar）
- [ ] 记录完整启动命令序列（apt → urdf_to_sdf → gz sim → RViz）
- [ ] 推导并记录差速运动学公式（v、ω ↔ 轮速）
- [ ] 整理"仿真到真机"会遇到的 3 个差异（预告 W15 DIY/真机选修）
