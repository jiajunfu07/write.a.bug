# W06 · Launch 文件 + 项目一（课件）

> 对应阅读: S Ch9
> 里程碑: **项目一 · 多节点动态系统** + 阶段考 1
> 配套笔记: 待写（建议命名 `Chapter-6-Launch-Files.md`）

## 学习目标

1. 用 Python launch 文件一键启动整套系统；
2. 掌握 launch arguments、参数注入、remappings 三件套；
3. 按验收清单交付项目一。

## 核心概念

### 1. launch 文件 = "系统启动清单"

把"先起谁、后起谁、各自参数是什么、话题怎么改名"固化成一个文件：

```python
# <pkg>/launch/relay.launch.py
from launch import LaunchDescription
from launch.actions import DeclareLaunchArgument
from launch.substitutions import LaunchConfiguration
from launch_ros.actions import Node

def generate_launch_description():
    robot_name = DeclareLaunchArgument('robot_name', default_value='relay-01')
    return LaunchDescription([
        robot_name,
        Node(package='sensor_relay', executable='sensor_pub',
             name='sensor_pub',
             parameters=[{'robot_name': LaunchConfiguration('robot_name'),
                          'rate_hz': 1.0}]),
        Node(package='sensor_relay', executable='relay_srv',
             name='relay_srv',
             remappings={'/sensor_state': '/station/' + 'sensor_state'}),
    ])
```

### 2. 三件套

- **launch arguments**：`ros2 launch sensor_relay relay.launch.py robot_name:=station-B`，或 `--ros-args -p name:=value`。
- **参数注入**：`parameters=[{...}]` 里可以混用常量、`LaunchConfiguration`、`FindExecutable` 等 substitutions。
- **remappings**：把节点私有/全局话题名改到别的名字（`'~/out': '/custom'`、`'/a:=/b'` 两种写法）。

### 3. 常用配套

- 日志级别：`--ros-args -r *:=info`
- 条件启动：`IfCondition(LaunchConfiguration('use_sim'))`（W10 之后会大量用）
- `EventHandlers`（on_shutdown 清理资源）—— 了解即可。

### 4. launch 文件要能被 `ros2 launch` 找到

Python 包：`setup.py` 的 `data_files` 必须包含 `('share/' + package_name + '/launch', glob('launch/*.launch.py'))`；C++ 包：CMake `install(DIRECTORY launch/ ...)`。

## 动手步骤

1. 把 W3–W5 的节点（pub/sub/srv/action）整合进一个 `sensor_relay` 包；
2. 写 `relay.launch.py`：4 个节点一次拉起；
3. 用两个不同的 launch argument 配置跑（station-A / station-B），验证"同一套代码两种配置"；
4. 录 3 分钟演示视频（见项目一验收）。

## 项目一验收清单

- [ ] ≥4 个节点：publisher、subscriber、service server、action server
- [ ] ≥1 个自定义 `msg` + 1 个自定义 `srv`
- [ ] 全部节点参数化；`ros2 launch ... robot_name:=A` 与 `:=B` 输出可区分
- [ ] 一个 launch 文件一键启动；`ros2 node list` 4 个节点齐全
- [ ] 架构图（节点 + 话题/服务/action 连线）
- [ ] 3 分钟演示视频 + README（构建、启动、验证步骤）

## 阶段考 1 范围（45 分钟闭卷）

- topics/services/actions 适用场景题 ×3
- QoS 场景题 ×4（给场景选 QoS 组合并解释）
- `ros2` CLI 实操：node/topic/srv/action/param/launch
- 代码挑错：给一段 pub+sub 代码找出 3 个 bug（常见：QoS 失配、忘 spin、entry_points）

## 常见坑

- `ros2 launch pkg file.py` 报 "not found in package" → setup.py data_files 没注册 launch 目录。
- remap 写法混用：`{'from': 'to'}` 字典 与 `'/a:=/b'` 字符串两种语法不要混在一个列表里。
- launch 里 `parameters` 传了 Python 运行时对象（如 datetime）→ 必须用 substitutions。
- 一个包多个 executable 时，`name` 参数别忘了给，否则节点名冲突/话题名撞车。

## 自测

1. launch 文件能做的 4 件事（启动顺序/参数/remap/条件）？
2. `--ros-args -p a:=1` 和 `ros2 launch pkg f.py a:=1` 的区别？
3. remappings 的两种写法分别是什么？
4. launch 文件放在哪个目录、如何注册才能被 `ros2 launch` 找到？
5. 什么时候该用 `IfCondition`？

### 参考答案

1. 按顺序启动一组节点、注入参数、重命名话题、按条件启停（EventHandlers 处理关闭事件）。
2. 前者给"被启动的节点"传参数（等价于 -p），后者是 launch 层的 argument，需要 `LaunchConfiguration` 去引用；语义层级不同。
3. `{'private_name': 'global_name'}` 字典；`'/old:=/new'` 冒号字符串。
4. `share/<pkg>/launch/`，并在 setup.py data_files（或 CMake install）中注册。
5. 可选组件（仿真开关、调试节点、不同硬件后端）不想每次都手删时。

## 笔记清单（对照书本补写）

- [ ] 抄写 launch 文件最小模板 + 每个 API 的中文注释
- [ ] 画项目一架构图（自己画的，不是抄书）
- [ ] 记录 station-A/B 两次启动的输出差异对比
- [ ] 总结"一个 launch 文件 vs 五个终端手动启动"的工程价值
