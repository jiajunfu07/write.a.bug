# W01 · ROS 2 是什么 + 环境搭建（课件）

> 对应阅读: S Ch1 (p.3–12)、S Ch2 (p.13–31)；M Ch1 (p.3–28)
> 里程碑: 跑通 turtlesim · 交付: 环境安装笔记 + 截图
> 配套笔记: 本站尚无（本课为新起点）

## 学习目标

1. 用自己的话说清 ROS 2 是什么、不是什么是（它不是操作系统）；
2. 画出 ROS 2 三层架构图，说清每层各举一个例子；
3. 解释 ROS 1 → ROS 2 的 5 个关键差异（DDS 是核心）；
4. 在 Ubuntu 24.04 上装好 ROS 2 Jazzy 并跑通 turtlesim。

## 核心概念

### 1. ROS 是什么

ROS = 一套**开源工具 + 库 + 社区**的机器人软件框架，提供：进程间通信机制（topic/service/action）、包管理（colcon/ament）、设备抽象、大量可复用算法包。
关键点：ROS 本身不运行在"它自己的系统"上，而是跑在 **Linux / RTOS / Windows** 之上 —— 所以说"ROS 不是 OS"。

### 2. 三层架构

```
┌────────────────────────────────────────────┐
│ 应用层   节点(node)、ros2 CLI、RViz、工具    │
├────────────────────────────────────────────┤
│ 中间件层  DDS 发布/订阅 + 发现 + QoS         │
├────────────────────────────────────────────┤
│ OS 层    Ubuntu / RTOS / ...                │
└────────────────────────────────────────────┘
```

- 应用层：你写的节点、`ros2 topic list`、RViz。
- 中间件层：DDS 负责节点间**自动发现 + 数据传输**（无需中心 broker）。
- OS 层：提供进程、线程、文件系统。

### 3. DDS 为什么重要（M Ch1 重点）

DDS（Data Distribution Service，OMG 标准）：**去中心化**的发布/订阅中间件 —— 每个进程自带"代理"，互相直接通信（peer-to-peer），自带 QoS（可靠性/时效/历史）。对比 ZeroMQ：ZMQ 没有标准发现机制和标准 QoS，跨厂商互通差。
ROS 2 通过 **RMW**（ros2 中间件抽象层）+ **RCL**（客户端库层）把 DDS 细节屏蔽掉，常用实现：`rmw_fastrtps_cpp`、`rmw_cyclonedds_cpp`（`echo $RMW_IMPLEMENTATION` 查看）。

### 4. 版本选择

Jazzy Jalisco = **LTS**，支持 Ubuntu 24.04，维护到 2029 年。非 LTS 版 18 个月就停更 → 课程/生产一律选 LTS。

## 动手步骤

1. **系统**：安装 Ubuntu 24.04（双系统 / VM；步骤见 S Ch2 p.16–24）。
2. **装 ROS 2**（官方仓库方式，对应 S Ch2 p.25–27）：

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install software-properties-common -y
sudo add-apt-repository universe -y
sudo apt update && sudo apt install curl -y
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
     -o /usr/share/keyrings/ros-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] \
http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" \
     | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
sudo apt update && sudo apt install ros-jazzy-desktop -y   # 想要精简就装 ros-jazzy-ros-base
```

3. **环境生效**：

```bash
echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc && source ~/.bashrc
```

4. **跑 turtlesim**（验证安装）：

```bash
sudo apt install ros-jazzy-turtlesim ros-jazzy-teleop-twist-keyboard ros-jazzy-rqt-graph -y
ros2 run turtlesim turtlesim_node
# 另开终端
ros2 run turtlesim turtle_teleop_key
ros2 topic list
ros2 topic echo /turtle1/cmd_vel
```

5. **多环境隔离**（为 W2 Docker 实验铺路）：

```bash
export ROS_DOMAIN_ID=1        # 取值 0–10，不同 ID 的两个 ROS 网络互不可见
```

## 常见坑

- 新终端 `ros2: command not found` → 忘了 `source`（检查 `~/.bashrc`）。
- 两个终端节点互不可见 → `ROS_DOMAIN_ID` 不一致，或 `RMW_IMPLEMENTATION` 不一致。
- VM 里两台"主机"通信失败 → NAT 网络限制；换 host-only / 桥接网卡（M Ch2 有详细讲多机配置）。
- 中文系统 locale 报错 → `export LANG=en_US.UTF-8`（S Ch2 p.18 有讲）。
- `ros2 topic list` 一直为空 → 确实没节点在跑，或 DDS 网卡配置（`CYCLONEDDS_URI` 指定网卡）没对上。

## 自测

1. 列出 ROS 2 三层架构，每层给一个组件例子。
2. 为什么说"ROS 不是操作系统"？它运行在哪一层之上？
3. DDS 与 ZeroMQ/消息队列 broker 的本质区别（提示：中心化 vs 去中心化、发现、QoS）？
4. `ROS_DOMAIN_ID=5` 的作用是什么？取值范围？
5. Jazzy 的维护周期到什么时候？为什么课程选它？

### 参考答案

1. 应用层（节点、RViz、ros2 CLI）/ 中间件层（DDS，经 RMW 抽象）/ OS 层（Ubuntu）。
2. ROS 是框架：库 + 工具 + 通信机制，运行在 Linux/RTOS 之上，不提供进程/内存管理。
3. DDS 是去中心化 pub/sub + 自动发现 + 标准 QoS；broker 模式需要中心代理，无标准发现/QoS 语义。
4. 隔离 DDS 网络：不同 ID 的节点互相"看不见"；0–10。
5. 2029 年；LTS，与 Ubuntu 24.04 配套，教程与生态最新。

## 笔记清单（对照书本补写）

- [ ] 用自己的话写"ROS 定义 + ROS equation"（M Ch1 p.4–7）
- [ ] 手绘三层架构图，标出 RMW/RCL 位置（M Ch1 p.24–26）
- [ ] 摘录 5 条 ROS1 vs ROS2 差异（M Ch1 p.12–13 对比表）
- [ ] 记录你的完整安装命令日志 + turtlesim 截图
- [ ] 写一段"我为什么选 Jazzy"（3 句话）
