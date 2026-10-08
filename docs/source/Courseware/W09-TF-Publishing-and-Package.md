# W09 · TF 发布与打包（课件）

> 对应阅读: S Ch12
> 里程碑: robot_description 包 + 一键 display
> 配套笔记: 待写（建议命名 `Chapter-9-TF-Publishing-and-Package.md`）

## 学习目标

1. 按标准布局组织 `robot_description` 包；
2. 打通 `joint_states → robot_state_publisher → /tf` 链路；
3. 一个 `display.launch.py` 在 RViz 中完整显示机器人。

## 核心概念

### 1. 标准包布局

```
robot_description/
├── urdf/      my_robot.urdf.xacro
├── config/    (rviz 配置、参数, 可选)
├── launch/    display.launch.py
└── rviz/      my_robot.rviz
```

### 2. 三个角色分工

| 角色 | 输入 | 输出 |
|---|---|---|
| joint_state_publisher(_gui) | 滑块 / 驱动信号 | `/joint_states` 话题 |
| robot_state_publisher | `robot_description`(URDF) + `/joint_states` | `/tf`（link 间变换） |
| RViz RobotModel | `robot_description` + `/tf` | 三维显示 |

**谁发谁**：
- link 之间的**固定**变换：来自 URDF，由 robot_state_publisher 算出并发布为 static TF；
- **关节**变换：由驱动（仿真控制器 / 真机驱动）以 `/joint_states` 频率发布，RSP 消费后转成 TF。
- map→odom、odom→base_link 这类**非 URDF 的 TF**：由定位/里程计节点发布（W10 之后）。

### 3. robot_state_publisher 最小用法

```python
Node(package='robot_state_publisher', executable='robot_state_publisher',
     parameters=[{'robot_description': open(urdf_path).read()}])
```

（launch 里更常见的是用 `Command` + `xacro` substitution 把 xacro 文件展开后注入。）

### 4. display.launch.py 骨架

```python
from launch import LaunchDescription
from launch_ros.actions import Node
from launch_ros.substitutions import FindPackageShare
from launch.substitutions import PathJoinSubstitution

def generate_launch_description():
    pkg_share = FindPackageShare('robot_description')
    urdf = PathJoinSubstitution([pkg_share, 'urdf', 'my_robot.urdf.xacro'])
    return LaunchDescription([
        Node(package='robot_state_publisher', executable='robot_state_publisher',
             parameters=[{'robot_description': Command(['xacro ', urdf])}]),
        Node(package='joint_state_publisher_gui', executable='joint_state_publisher_gui',
             parameters=[{'source_list': ['joint_states']}]),
        Node(package='rviz2', executable='rviz2',
             arguments=['-d', PathJoinSubstitution([pkg_share, 'rviz', 'my_robot.rviz'])]),
    ])
```

## 动手步骤

1. 把 W08 的 xacro 移入 `robot_description/urdf/`，补齐 package.xml 依赖：`robot_state_publisher`、`joint_state_publisher`、`joint_state_publisher_gui`、`xacro`、`rviz`。
2. setup.py `data_files` 注册 `urdf/`、`launch/`、`rviz/` 三个目录。
3. `colcon build` → `ros2 launch robot_description display.launch.py` → RViz 中滑块拖动关节（若有）观察 TF 变化。
4. `ros2 topic hz /joint_states`、`ros2 run tf2_tools view_frames` 验证链路。

## 常见坑

- `robot_description` 参数没给 → RSP 启动即退出（看日志）。
- setup.py 没注册 `urdf/` 目录 → `FindPackageShare` 找不到文件，launch 报 substitution 失败。
- RViz 里 RobotModel 显示但 **TF 不显示** → 检查 `/tf` 话题与 Fixed Frame。
- `joint_state_publisher_gui` 与真实驱动同时发 `/joint_states` → 话题被"抢"，运动抖动；真机环境去掉 GUI。
- xacro 展开失败（路径/属性错）→ `Command(['xacro ', ...])` 的报错常被包在 substitution 错误里，先手动 `xacro file.xacro` 跑一遍。

## 自测

1. `/tf` 里的 link 变换是谁发布的？数据源是什么？
2. joint_state_publisher_gui 什么时候用、什么时候必须去掉？
3. `robot_description` 话题的内容是什么格式？
4. 为什么要把模型放进独立包而不是直接散在工作区？
5. W10 起 Gazebo 需要哪些"额外输入"才能动起来？（预告：ros2_control / diff drive）

### 参考答案

1. robot_state_publisher；数据源 = URDF（结构）+ `/joint_states`（关节角）。
2. 仿真调试/无驱动时用来手动给关节角；真机或仿真控制器已发布 joint_states 时必须去掉，避免话题冲突。
3. 展开后的 URDF 文本（string 消息）。
4. 可复用（launch/仿真/导航都要引用）、依赖清晰、版本管理、避免"散落文件"。
5. 差速控制器（cmd_vel→轮速）、轮子 continuous joint 的状态发布、里程计。

## 笔记清单（对照书本补写）

- [ ] 画 "joint_states → RSP → /tf → RViz" 数据流图
- [ ] 抄写 display.launch.py 模板 + 注释每个 API
- [ ] 记录 robot_description 包的完整目录树 + 每个文件职责
- [ ] 总结 launch 中 substitutions 的 5 个常用（FindPackageShare / PathJoin / Command / LaunchConfiguration / OpaqueFunction）
