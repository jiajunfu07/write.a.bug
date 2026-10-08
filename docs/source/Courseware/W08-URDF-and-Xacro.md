# W08 · URDF / Xacro（课件）

> 对应阅读: S Ch11；M Ch4 (p.119–176, URDF/Xacro 部分)
> 里程碑: 差速底盘 + LiDAR 模型（xacro 版）
> 配套笔记: 待写（建议命名 `Chapter-8-URDF-and-Xacro.md`）

## 学习目标

1. 掌握 URDF 四大件：link / joint / inertial / visual+collision；
2. 写一个能通过 `check_urdf`、能在 RViz 中显示的差速底盘；
3. 用 xacro 的 properties + macro 消除重复。

## 核心概念

### 1. URDF 是什么

URDF = **静态结构**描述：由哪些 link 组成、用什么 joint 连接、每个 link 长什么样、有多重。它**不描述**运动学参数、传感器数据流、控制器（这些在 SDF / ros2_control / 包里）。

### 2. 一个最小完整 link

```xml
<link name="base_link">
  <visual>
    <geometry><box size="0.4 0.3 0.15"/></geometry>
    <origin xyz="0 0 0.075" rpy="0 0 0"/>
    <material name="grey"><color rgba="0.5 0.5 0.5 1"/></material>
  </visual>
  <collision>
    <geometry><box size="0.4 0.3 0.15"/></geometry>
    <origin xyz="0 0 0.075" rpy="0 0 0"/>
  </collision>
  <inertial>
    <mass value="5.0"/>
    <origin xyz="0 0 0.075" rpy="0 0 0"/>
    <inertia ixx="0.04" ixy="0" ixz="0" iyy="0.04" iyz="0" izz="0.04"/>
  </inertial>
</link>
```

要点：
- **units 是 SI**：米、千克、弧度；坐标系**右手系、z 向上**。
- `<origin>` 先平移 `xyz` 再旋转 `rpy`（rpy 是绕 X→Y→Z 的固角）。
- visual 给渲染、collision 给物理、inertial 给动力学 —— **三者原点要对得上**，不然仿真里机器人"歪着摔跤"。

### 3. joint 类型

| type | 含义 | 典型 |
|---|---|---|
| fixed | 刚性连接 | 底座装激光雷达 |
| revolute | 有上下限的旋转 | 机械臂肩/肘 |
| continuous | 无上下限的旋转 | 差速轮、万向轮 |
| prismatic | 直线 | 液压杆 |

```xml
<joint name="left_wheel_joint" type="continuous">
  <parent link="base_link"/>
  <child link="left_wheel_link"/>
  <origin xyz="0.05 0.18 -0.03" rpy="0 0 0"/>
  <axis xyz="0 1 0"/>
</joint>
```

### 4. Xacro = URDF 的"宏 + 变量"

```xml
<xacro:properties>
  <xacro:property name="wheel_radius" value="0.05"/>
  <xacro:property name="wheel_width"  value="0.03"/>
</xacro:properties>

<xacro:macro name="wheel" params="prefix offset_y">
  <link name="${prefix}_wheel_link"> ... </link>
  <joint name="${prefix}_wheel_joint" type="continuous"> ... </joint>
</xacro:macro>

<xacro:wheel prefix="left"  offset_y=" 0.18"/>
<xacro:wheel prefix="right" offset_y="-0.18"/>
```

支持 `${math}`：`<xacro:property name="inertia" value="${1/12 * mass * (lx*lx + lz*lz)}"/>`。

### 5. 验证工具链

```bash
check_urdf -p my_robot.urdf.xacro     # 语法/结构检查
ros2 run robot_state_publisher robot_state_publisher \
     _description_file:=$(pwd)/my_robot.urdf.xacro
ros2 run rviz2 rviz2                  # RobotModel 显示
```

## 动手步骤

1. 先写**纯 URDF**：base_link + 2 轮 + 1 个 lidar_link（fixed joint），`check_urdf` 通过。
2. RViz 中显示并截图（Fixed Frame = base_link）。
3. 重构为 **xacro**：轮子抽 macro、半径/宽度抽 property，确认渲染结果不变（对比截图）。
4. 挑战（M Ch4 风格）：给 base 加 caster（万向轮）或把 lidar 高度参数化。

## 常见坑

- **忘 inertial** → 能渲染、能 check，但一进 Gazebo 就穿地/爆炸。
- joint axis 与视觉朝向不一致 → 轮子"视觉向上滚，实际绕 z 转"，机器人漂移。
- visual 与 collision 尺寸/原点不一致 → 碰撞体积和看到的模型对不上。
- xacro 属性名拼错不报错（`${undefined_var}` 直接原样输出）→ 渲染出怪东西；养成 `check_urdf` + RViz 双验证。
- 单位错：轮子半径写成 50（cm）→ 机器人"巨无霸"。
- link/joint 名重复、或 joint 的 parent/child 指错 → 树不成形。

## 自测

1. URDF 的四大组成部分？
2. `type="continuous"` 与 `type="revolute"` 的区别？各举一例。
3. `<origin xyz rpy>` 的应用顺序是什么？
4. visual / collision / inertial 三个块分别给谁用？
5. 什么时候必须用 xacro 而不是纯 URDF？

### 参考答案

1. link、joint、inertial、visual/collision（material 属 visual）。
2. continuous 无角度限位（轮子、云台）；revolute 有 lower/upper 限位（机械臂关节）。
3. 先平移 xyz，再绕 X→Y→Z 依次旋转 rpy。
4. visual→渲染（RViz/Gazebo 图形）；collision→物理引擎碰撞检测；inertial→动力学（质量+转动惯量）。
5. 有重复结构（多个轮子/夹爪）、需要参数化（半径、轴距）或计算派生值（惯量公式）时。

## 笔记清单（对照书本补写）

- [ ] 抄写最小 link 模板 + 每个标签的中文注释
- [ ] 画自己机器人的 URDF 结构树（link/joint）
- [ ] 记录 `check_urdf` 一次真实报错 + 原因
- [ ] 写 xacro 重构前后对比（行数、参数化能力）
- [ ] 整理"URDF 能做什么 / 不能做什么"两栏清单
