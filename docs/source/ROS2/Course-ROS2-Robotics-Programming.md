# ROS 2 Robotics Programming — 课程大纲与学习计划

> **课程编号**: ROS2-301 · **学分**: 3 · **周期**: 16 周（每周 2 讲时 + 4 实验时，共 96 学时）
> **面向人群**: 计算机 / 自动化 / 机器人专业本科生，以及系统自学者
> **教材**:
> - 主教材 **M** — *Mastering ROS 2 for Robotics Programming*, 4th ed.（Packt, 2025-07, 576 页），Lentin Joseph & Jonathan Cacace
> - 辅教材 **S** — *ROS 2 from Scratch*（Packt, 2024-11, 380 页），Edouard Renard

## 1. 课程概述

本课程以"两本教材、一条主线"组织：**S 书负责从 0 到 1 打基础**（概念、通信、建模、仿真），**M 书负责从 1 到 N 做系统**（控制、导航、操作、感知、LLM、强化学习）。

### 学习产出（学完你能够）

1. 独立搭建 ROS 2（Jazzy）开发环境（裸机 / 虚拟机 / Docker），并用 `colcon` 开发构建 Python 与 C++ 节点；
2. 用 topics、services、actions、parameters 和 QoS 设计节点间通信系统；
3. 用 URDF/Xacro 描述机器人模型、发布 TF，并加载到 RViz 与 Gazebo；
4. 搭建"SLAM 建图 + Nav2 自主导航"完整系统，并用 Behavior Tree 编排机器人任务序列；
5. 用 MoveIt 2 完成多自由度机械臂的轨迹规划；
6. 理解 ros2_control 控制框架并对接（仿真）硬件接口；
7. 选修方向：LLM 集成 / 深度强化学习 / 空中机器人 / 测试与 CI-CD 中至少完成一个；
8. 交付一个完整的机器人项目：含架构设计、代码、文档与答辩演示。

### 先修要求

- Linux 命令行（自动补全、文件系统、`.bashrc` 与环境变量）
- Python 3（面向对象）
- C++（基础即可；只想走 Python 路线可暂时跳过 C++ 部分）
- Git
- 无需 ROS / ROS 1 经验

## 2. 两本书怎么配合

| 维度 | S — ROS 2 from Scratch | M — Mastering ROS 2 (4th) |
|---|---|---|
| 定位 | 入门主线，Python 优先，循序渐进，每章带练习 | 体系化进阶，项目导向，覆盖导航 / 操作 / ML |
| 适合 | 概念讲解、基础实验（W1–W10） | 进阶专题、里程碑项目、毕业项目（W11–W16） |
| 深度 | "把它跑起来" | "理解为什么 + 做成系统" |

**阅读规则**：同一主题先读 S 建立直觉，再读 M 加深；S 到 Gazebo（Ch13）收尾后，进入 M 的应用层。

| 课程主题 | S 章节 | M 章节 |
|---|---|---|
| ROS 2 概念与架构 | Ch1 (p.3–12) | Ch1 (p.3–28) |
| 安装与环境 | Ch2 (p.13–31) | Ch2 (p.31–72) |
| 第一个节点 | Ch3–Ch4 | Ch3 (p.75–117) |
| Topics / Services / Actions / Parameters | Ch5–Ch8 | Ch3 (p.75–117) |
| Launch 文件 | Ch9 | Ch3 |
| TF 与 RViz | Ch10 | — |
| URDF / Xacro | Ch11 | Ch4 (p.119–176) |
| TF 发布与打包 | Ch12 | Ch4 |
| Gazebo 仿真 | Ch13 | Ch5 (p.177–214) |
| ros2_control | — | Ch6 (p.217–242) |
| Behavior Tree | — | Ch7 (p.243–268) |
| Nav2 + SLAM | — | Ch8 (p.269–305) |
| MoveIt 2 | — | Ch9 (p.307–330) |
| 感知栈 | — | Ch10 |
| 空中机器人（选修） | — | Ch11 |
| DIY 移动机器人（选修） | — | Ch12 |
| 测试与 CI/CD（选修） | — | Ch13 (p.419–442) |
| LLM 集成（选修） | — | Ch14 (p.443 起) |
| 深度强化学习（选修） | — | Ch15 |
| 可视化 / 仿真插件（选修） | — | Ch16 |

## 3. 16 周总表

| 周 | 模块 | 主题 | 主要阅读 | 里程碑 |
|---|---|---|---|---|
| 1 | M1 基础 | ROS 2 是什么 + 环境安装 | S Ch1–2；M Ch1 | 跑通 turtlesim |
| 2 | M1 基础 | 第一个节点（Python / C++） | S Ch3–4；M Ch3 | P0: greeting 节点 |
| 3 | M2 通信 | Topics 与 QoS | S Ch5；M Ch3 | 自定义消息 |
| 4 | M2 通信 | Services + Parameters | S Ch6, Ch8 | 运行时动态配置 |
| 5 | M2 通信 | Actions | S Ch7 | action 反馈 / 取消 |
| 6 | M2 通信 | Launch + 阶段考 1 | S Ch9 | **项目一** |
| 7 | M3 建模仿真 | TF 与 RViz | S Ch10 | TF 树演示 |
| 8 | M3 建模仿真 | URDF / Xacro | S Ch11；M Ch4 | 机器人模型 |
| 9 | M3 建模仿真 | TF 发布 + 打包 | S Ch12 | robot_description 包 |
| 10 | M3 建模仿真 | Gazebo + 阶段考 2 | S Ch13；M Ch5 | **项目二** |
| 11 | M4 应用 | ros2_control | M Ch6 | 控制器演示 |
| 12 | M4 应用 | Behavior Tree + Nav2 | M Ch7–8 | 导航演示 |
| 13 | M4 应用 | MoveIt 2 | M Ch9 | 机械臂规划演示 |
| 14 | M4 应用 | 感知 / LLM（选修） | M Ch10, Ch14 | 感知或 LLM 演示 |
| 15 | M5 综合 | 毕业项目开题 + 选修 | M Ch11/12/13/15 | 开题评审 |
| 16 | M5 综合 | 毕业项目答辩 | — | **最终项目** |

## 4. 分模块详细计划

> 每周的**课件**见侧边栏 "ROS 2 Courseware" 分区（`Courseware/` 目录）：对照书本阅读 + 按课件"动手步骤"做实验，再用每页末尾的"笔记清单"补齐本章笔记。

### 模块一 · 基础（W1–W2）

#### W1 · ROS 2 是什么
- **目标**：说清楚 ROS 2 三层架构、ROS 1 vs ROS 2 差异、DDS 的角色；完成开发环境安装。
- **阅读**：S Ch1 (p.3–12)、S Ch2 (p.13–31)；M Ch1 (p.3–28)
- **讲解要点**：ROS equation；OS / 中间件 / 应用三层与 RMW、RCL；DDS 与 `ROS_DOMAIN_ID`；版本选择（Jazzy LTS）。
- **实验**：① 安装 Ubuntu 24.04（VM 或双系统）+ ROS 2 Jazzy；② `source` 写入 `.bashrc`；③ `ros2 topic list` 验证 + `turtlesim` 与键盘遥操作。
- **交付物**：环境安装笔记（提交到仓库 `docs/`）+ turtlesim 运行截图。
- **自检**：node 与进程的区别？为什么 ROS 2 用 DDS 而不是 ZeroMQ？`ROS_DOMAIN_ID` 起什么作用？

#### W2 · 开发工具与第一个节点
- **目标**：用 `colcon` 创建 / 构建工作区，分别用 Python 和 C++ 写出第一个节点。
- **阅读**：M Ch2 (p.31–72，重点 Docker / Dev Container / domain ID)、S Ch3（核心概念）、S Ch4（编写节点）
- **实验**：① 建 `hello_ros2` 工作区：Python 节点（定时发布问候）+ C++ 节点（订阅并记录）；② `ros2 node list/info`、`rqt_graph` 观察；③（选做）把两个节点放进两个容器 —— 见本站 Chapter-0。
- **交付物**：repo + 2 个节点 + README（构建与运行方法）。
- **自检**：`setup.py entry_points` 的作用？`ros2 run` 与 `ros2 launch` 的区别？Python 包与 C++ 包在 colcon 中的差异？

### 模块二 · 节点间通信（W3–W6）

#### W3 · Topics
- **目标**：掌握发布 / 订阅、自定义消息、QoS 基础。
- **阅读**：S Ch5；M Ch3（topics 相关小节）
- **实验**：① `std_msgs` 发布 / 订阅；② 定义自定义 `msg`（如 `SensorState`）；③ QoS 演示：`transient_local` 与 `volatile` 的差异（后上线的订阅者能否拿到最新值）。
- **自检**：pub/sub 与 client/server 的区别？说出 4 种 QoS 策略及各自适用场景。

#### W4 · Services + Parameters
- **阅读**：S Ch6、S Ch8
- **实验**：① 自定义 `srv`（如 `SetTarget`）+ client / server；② 节点声明 ≥3 个参数 + 参数回调（运行时改参数，行为随之变化）；③ `ros2 param get/set` 验证。
- **自检**：什么情况下用 service 而不是 action？参数和 topic 的本质区别？

#### W5 · Actions
- **阅读**：S Ch7
- **实验**：① 实现 Fibonacci 风格 action（feedback + result）；② 取消运行中的 goal；③ 画出 action 的 goal / result / feedback 三通道时序图。
- **自检**：为什么 service 表达不了"长耗时 + 可取消 + 有反馈"的任务？

#### W6 · Launch + 阶段考 1（项目一）
- **阅读**：S Ch9
- **项目一 · 多节点动态系统**（示例：传感器中转站），要求：
  - ≥4 个节点：publisher、subscriber、service server、action server
  - ≥1 个自定义 `msg` + 1 个自定义 `srv`
  - 全部节点可用参数配置，且同一套代码支持两套不同配置
  - 一个 `ros2 launch` 一键启动；附架构图 + 3 分钟演示视频
- **阶段考 1**（闭卷 45 分钟）：topics / services / actions 的适用场景；4 个 QoS 场景题；`ros2` CLI（node / topic / srv / action / param / launch）实操；一段代码挑错。
- **自检**：launch 文件中 remappings 与 launch arguments 的作用？

### 模块三 · 建模与仿真（W7–W10）

#### W7 · TF 与 RViz
- **阅读**：S Ch10
- **实验**：① 手动发布 TF（base → gripper）；② RViz 中查看（Fixed Frame / RobotModel / TF 显示）；③ tf2 listener 查询指定时刻的变换。
- **自检**：为什么需要"坐标树"而不是单一坐标系？static transform 用在什么场合？

#### W8 · URDF / Xacro
- **阅读**：S Ch11；M Ch4 (p.119–176，URDF / Xacro 部分)
- **实验**：① 写差速底盘 + 2D LiDAR 的 URDF；② 重构为 xacro（轮子抽成 macro）；③ `check_urdf` 校验 + RViz 中查看。
- **自检**：`<joint>` 的 type 与 `<inertial>` 各自的作用？Xacro 解决了 URDF 的什么问题？

#### W9 · TF 发布与打包
- **阅读**：S Ch12
- **实验**：① 打包为 `robot_description`（urdf / launch / rviz config）；② `joint_state_publisher` + `robot_state_publisher` 打通 TF 树；③ 一个 `display.launch.py` 一键打开。
- **自检**：link 之间的 TF 由谁发布？joint 的 TF 又由谁发布？

#### W10 · Gazebo + 阶段考 2（项目二）
- **阅读**：S Ch13；M Ch5 (p.177–214)
- **实验**：① URDF 添加 Gazebo 插件标签；② Gazebo 中 spawn 并用 `cmd_vel` 驱动；③ 增加一个传感器（相机或 LiDAR）并在 RViz 中查看数据。
- **项目二 · 自定义机器人仿真**：把 W8–W9 的机器人在 Gazebo 中跑起来 —— 话题驱动、发布里程计、传感器出数据；3 分钟演示视频。
- **阶段考 2**：TF 树结构 / URDF 关节与惯量 / Gazebo 架构（server & client）/ 差速运动学。
- **自检**：Gazebo Sim 的 server 与 client 分别负责什么？物理引擎在其中承担什么角色？

### 模块四 · 机器人应用（W11–W14）

#### W11 · ros2_control
- **阅读**：M Ch6 (p.217–242)
- **实验**：① 配置 `controller_manager` + `diff_drive_controller`；② `ros2 control` CLI 管理控制器；③（进阶）实现一个简单的前向控制器插件。
- **自检**：ros2_control 的四大件（硬件接口 / 控制器 / controller manager / 插件）各自职责？

#### W12 · Behavior Tree + Nav2
- **阅读**：M Ch7 (p.243–268)、M Ch8 (p.269–305)
- **实验**：① 用 Groot2 编写 BT（sequence / fallback / decorator）；② 跑通 Nav2 演示（TurtleBot3 或自己的机器人）；③ SLAM Toolbox 建图 + 在地图上自主导航；④（进阶）定制 BT Navigator。
- **自检**：Nav2 为什么用行为树而不是写死状态机？

#### W13 · MoveIt 2
- **阅读**：M Ch9 (p.307–330)
- **实验**：① 为 W8/W9 的机械臂配置 MoveIt 2；② RViz 中规划 + 添加障碍物；③ 编写调用 `move_group` 的规划节点（Python 或 C++）；④（进阶）笛卡尔路径规划。
- **自检**：SRDF 与 planning scene 分别起什么作用？

#### W14 · 感知 / LLM（选修，二选一）
- **感知路线**：M Ch10 —— camera pipeline（`camera_info`、点云）、与 OpenCV / PCL 的基本集成。
- **LLM 路线**：M Ch14 (p.443 起) —— LLM agent 节点：接入大模型，通过 tools / actions 控制 turtlesim。
- **自检**：传感器带时间戳的数据如何与决策层对齐？

### 模块五 · 进阶与综合（W15–W16）

#### W15 · 毕业项目开题
- **选修深化**（任选其一或多）：M Ch11（空中机器人）/ Ch12（DIY 移动机器人）/ Ch13（测试与 CI-CD）/ Ch15（深度强化学习）
- **交付物**：开题报告（1 页：目标、架构图、里程碑、风险）+ 开题评审。

#### W16 · 毕业项目答辩
- 10 分钟演示 + 10 分钟问答；提交 repo + 项目报告 + 演示视频。

## 5. 考核方式

| 项目 | 占比 | 说明 |
|---|---|---|
| 每周实验 | 20% | 按各周"交付物"打分，取平均 |
| 阶段考 | 10% | 两次（W6、W10），各 5% |
| 项目一 | 15% | W6 多节点动态系统 |
| 项目二 | 20% | W10 自定义机器人仿真 |
| 毕业项目 | 25% | W16，含答辩 |
| 平时表现 | 10% | 代码互评、提问与答疑、学习日志 |
| **合计** | **100%** | ≥60 分及格 |

- **迟交**：每天 −10%，超过 3 天不接受。
- **学术诚信**：禁止整段抄用他人代码；引用开源代码必须标注来源与许可证。

## 6. 毕业项目选题（任选其一）

| 编号 | 项目 | 核心栈 | 必做 | 加分项 |
|---|---|---|---|---|
| A | 自主导航服务机器人 | Nav2 + SLAM Toolbox + BT | 自定义 URDF、建图、自主导航、BT 任务链（导航 → 汇报 → 返回） | 动态避障、多航点 |
| B | 机械臂搬运 | MoveIt 2 +（可选）感知 | 规划到固定位姿、避障、笛卡尔抓取 | 视觉位姿估计 |
| C | LLM 任务调度 | M Ch14 + A/B 之一 | 自然语言任务 → action 链、工具调用 | 记忆与多轮对话 |
| D | DRL 导航策略 | M Ch15 + Gymnasium | 仿真训练、与 Nav2 对比 | 域随机化、真机部署 |
| E | DIY 移动机器人 | M Ch12 + 硬件（可选） | RPi + 差速底盘 + LiDAR、里程计 | 实车 SLAM |

**评分量规（rubric）**：功能完整性 40% / 代码质量与架构 25% / 文档与演示 20% / 创新性 15%。

## 7. 自学模式（Self-study Adaptation）

独自学习时没有"学生 / 教师"分工，改用"里程碑 + 自检"模式：

1. **验收门**：上一里程碑的交付物不通过，不进入下一模块（用每节的"自检"问题 + 验收清单自测）。
2. **标准节奏（16 周）**：每周 10–12 小时 —— 3 小时阅读与笔记（等价讲时）、4 小时实验、其余用于项目与复习。
3. **紧凑节奏（8 周）**：每周 20–24 小时，按两周合并（W1-2 / W3-4 / W5-6 / W7-8 / W9-10 / W11-12 / W13-14 / W15-16）。
4. **学习日志**：每周笔记沿用本站既有章节笔记格式（英文 + `.zh.md` 中文对照），提交到本仓库。
5. **模拟答辩**：W16 录一段演示视频，对照 rubric 自评；低于 80 分则返工一次。

## 8. 环境清单

- [ ] Ubuntu 24.04（裸机 / 双系统 / 虚拟机）—— 或 Docker + Dev Container（见本站 Chapter-0）
- [ ] ROS 2 Jazzy（LTS）+ `ros-jazzy-demo-common`、`ros-jazzy-turtlesim`
- [ ] VS Code + C/C++、Python、ROS 扩展
- [ ] 16 GB 内存（Gazebo / RViz 较吃内存）；DRL / Isaac Sim 选修需 GPU + CUDA
- [ ]（可选）Raspberry Pi 4/5 + 电机驱动 + 2D LiDAR（W15 硬件选修）
- [ ]（可选）LLM API key（W14 LLM 选修）

## 9. 与本站已有章节的对应（学习进度）

| 周 | 本站章节笔记 | 对应书章节 | 状态 |
|---|---|---|---|
| W1–W2 | `Chapter-0-Run-2-nodes-in-single-or-separate-docker-containers.md`、`Chapter-1-Writing-and-Building-a-ROS-2-Node.md` | S Ch2–4 | 已写 |
| W3 | `Chapter-2-Topics-Sending-and-Receiving-Messages-between-Nodes.md` | S Ch5 | 已写 |
| W4 | `Chapter-3-Service-Client-Server-Interation-between-Nodes.md`、`Chapter-5-Parameters-Making-Nodes-More-Dynamic.md` | S Ch6、Ch8 | 进行中 |
| W5 | — | S Ch7（actions） | 待写 |
| W6 | — | S Ch9（launch） | 待写 |
| W7–W10 | — | S Ch10–13；M Ch4–5 | 待写 |
| W11–W14 | — | M Ch6–10 | 待写 |
| W15–W16 | — | M Ch11–16（选修） | 待写 |

> 建议按课程周次顺序补写章节笔记，每完成一章即可在大纲上打勾，形成"写作即学习"的闭环。

## 10. 补充资源

- ROS 2 官方文档（Jazzy）: https://docs.ros.org/jazzy/
- Nav2: https://navigation.ros.org/ · MoveIt 2: https://moveit.picknik.ai/ · ros2_control: https://control.ros.org/
- 官方演示包：`ros-jazzy-turtlesim`、`ros-jazzy-demo-nodes-cpp/py`
- S 书示例代码: https://github.com/PacktPublishing/ROS-2-from-Scratch
- 本站已有章节笔记：本仓库 `docs/source/ROS2/`
