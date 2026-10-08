# W13 · MoveIt 2（课件）

> 对应阅读: M Ch9 (p.307–330)
> 里程碑: 机械臂规划演示（含障碍物）
> 配套笔记: 待写（建议命名 `Chapter-13-MoveIt2.md`）

## 学习目标

1. 说清 MoveIt 2 的组件（move_group、planning pipeline、SRDF、RViz 插件）；
2. 用 Setup Assistant 为自己的手臂生成配置包；
3. 在 RViz 中做笛卡尔/关节空间规划，并加障碍物；
4. 写一个调用 move_group 的规划节点（Python 或 C++）。

## 核心概念

### 1. MoveIt 2 是什么

"操作栈"：给机械臂/手提供**运动规划 + 碰撞检测 + 环境管理**。核心组件：

| 组件 | 职责 |
|---|---|
| move_group 节点 | 规划服务入口（action/service 接口） |
| Planning Pipeline | OMPL 等规划器 + 运动学求解器（KDL）+ 碰撞检测 |
| SRDF | 语义层：group、end_effector、默认位姿、禁用碰撞对 |
| RViz 插件 | 可视化 + 交互规划 + 发布 planning scene |
| MoveIt Task Constructor (可选) | 多步任务组合（了解即可） |

### 2. SRDF 与 URDF 的关系

URDF = 几何/连接（W8 已学）；SRDF = **语义**：哪些关节组成一个 group、谁是末端、哪些碰撞对可以忽略（相邻 link 永远"碰着"不算碰撞）。

### 3. 规划两种空间

- **关节空间**：起点→终点的关节角插值（OMPL RRT*/PRM*），适合大范围。
- **笛卡尔空间**：末端沿直线/圆弧走（`set_cartesian_path` + `set_max_cartesian_error`），适合抓取轨迹。

### 4. 规划请求的关键参数（代码级）

```cpp
moveit::planning_interface::MoveGroupInterface arm("arm");
arm.setPlanningTime(2.0);
arm.setMaxVelocityScaling(0.5);
arm.setStartState(*arm.getCurrentState());
arm.setEndPosition({0.5, 0.0, 0.4});
arm.setPoseTarget(target_pose);          // 或 setEndPosition
arm.setPlanningPipelineID("OMPL");
// 加障碍物:
Eigen::Isometryd box = Eigen::Isometryd::Identity();
box.translate(Eigen::Vector3d(0.3, 0.2, 0.25));
arm.addBoxCollision("box", box, *arm.getPlanningFrame());
auto res = arm.plan();                   // 或 plan(motion_plan)
arm.execute(res);
```

Python 侧可用 `moveit`（moveit_py）或走 move_group 的 action/service（M Ch9 用 C++ 节点演示）。

## 动手步骤

1. `ros2 run moveit_setup_assistant moveit_setup_assistant` → 选自己的 URDF → 配 group/关节限制 → 生成 config 包（M Ch9 Step 1–2）。
2. RViz 中：MoveIt Motion Planning 面板 → 拖动末端 → 规划 → 播放。
3. 加障碍物（RViz "Add Box"）→ 观察路径绕行。
4. 写代码节点：给定点/位姿目标，调用 move_group 规划并执行（M Ch9 p.320–328 模板）。
5. （进阶）笛卡尔路径：`set_cartesian_path(..., 0.01)` 走一段直线抓取。

## 常见坑

- SRDF group 名与代码里 `MoveGroupInterface("arm")` 不一致 → 找不到 group。
- 规划一直失败 → 先查：目标位可达吗（工作空间）、碰撞 margin 是否过大、planning time 是否太短。
- 忘了 `setStartState`（当前状态变了）→ 轨迹从错误位姿出发。
- 仿真里"规划成功但执行撞了" → 碰撞检测模型与实际视觉/物理模型不一致，或执行中环境变化（需 re-plan）。
- RViz 的 RobotModel 不显示 → 同 W9 的 RSP 问题（MoveIt 依赖 robot_description + /tf）。

## 自测

1. URDF 与 SRDF 各负责什么？
2. 关节空间规划与笛卡尔规划分别适合什么场景？
3. `set_max_cartesian_error` 是什么？
4. "规划成功但执行中失败"的 3 个可能原因？
5. MoveIt 的规划器（OMPL）属于什么算法族？举一个（RRT*/PRM*/CHOMP）。

### 参考答案

1. URDF=几何/连接/惯性；SRDF=语义（group、末端、默认位姿、碰撞对豁免）。
2. 关节空间：大范围重定位、姿态调整；笛卡尔：需要末端走直线/圆弧（抓取、插拔、打磨）。
3. 笛卡尔路径每步允许的末端位置误差（米），步长越小越平滑但越可能失败。
4. 环境变化（新障碍物）、执行器跟随误差、planning scene 与实际不同步、速度/加速度超限被裁剪。
5. 采样型（基于图/随机）规划算法族；如 RRT*、PRM*（CHOMP 是轨迹优化）。

## 笔记清单（对照书本补写）

- [ ] 画 MoveIt 2 组件图（URDF/SRDF → move_group → RViz/代码）
- [ ] 记录 Setup Assistant 配置步骤（截图 3 张关键页）
- [ ] 抄写 C++ 规划节点模板 + 注释每个 API
- [ ] 整理"规划失败"排查清单（≥5 条）
- [ ] 对比一次关节空间 vs 笛卡尔规划的路径截图 + 结论
