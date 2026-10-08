# W12 · Behavior Tree + Nav2（课件）

> 对应阅读: M Ch7 (p.243–268)、M Ch8 (p.269–305)
> 里程碑: 建图 + 自主导航演示
> 配套笔记: 待写（建议命名 `Chapter-12-Behavior-Tree-and-Nav2.md`）

## 学习目标

1. 掌握 BT 的控制节点（Sequence/Fallback/Decorator）与叶子节点（Action/Condition）；
2. 用 Groot2 可视化并运行自己的 BT；
3. 理解 Nav2 的 server 架构与"为什么用 BT 编排"；
4. 完成 SLAM Toolbox 建图 + Nav2 导航闭环。

## 核心概念

### 1. Behavior Tree 基础

BT = 一棵**控制流树**，自顶向下 tick：

| 节点类型 | 角色 | 例子 |
|---|---|---|
| Sequence | 全部成功才成功（短路） | 先开门→再抓取 |
| Fallback | 任一成功即成功（试错） | 直行→绕行→后退 |
| Decorator | 修饰一个子节点 | 重试 N 次、倒序 |
| Action | 干活的叶子 | NavigateToPose |
| Condition | 判断的叶子 | 电量 > 10%? |

**Blackboard** = 节点间共享的键值存储（目标点、状态标志）。

为什么用 BT 而不是状态机：声明式、可视化（Groot2）、失败路径可组合、热替换不重启。

### 2. Groot2 上手

```bash
sudo apt install ros-jazzy-behavior-tree-cpp-v3-...  # 以 M Ch7 的安装清单为准
# 写 my_tree.xml (BT 的 XML 格式, M Ch7 p.250-254 有完整模板)
groot2-play my_tree.xml
```

### 3. Nav2 架构（M Ch8 核心）

```
                 ┌─────────────────────────┐
  action:        │  BT Navigator Server     │  ← BT 定义"任务 + 恢复策略"
  NavigateToPose │  Behavior Server         │  ← spin / backup / wait 等恢复行为
                 ├─────────────────────────┤
                 │  Planner Server (全局)   │
                 │  Controller Server(局部) │
                 │  Smoother / Waypoint Follower │
                 │  Velocity Smoother / Collision Monitor │
                 └─────────────────────────┘
        + Lifecycle Manager (统一 configure/activate)
```

- 任务都是 **action**：`NavigateToPose`、`ComputePathThroughPose`、`FollowWaypoints`、`ReturnToBase`。
- 导航依赖：`/tf`（map→odom→base）、`/odom`、`/scan`、`/map`。

### 4. 建图 → 导航流程

1. **建图**：SLAM Toolbox（2D LiDAR）→ 输出 `/map`（`nav_msgs/OccupancyGrid`）；
2. **存图**：RViz 的 "Publish map" 或 `ros2 topic echo /map --no-arr > map.yaml`（按 M Ch8 的工具链）；
3. **导航**：Nav2 加载 map + 定位（初始位姿由 RViz 或 action 给）→ `NavigateToPose`。

### 5. 手动发导航 action（验收用）

```bash
ros2 action send_goal /navigate_to_pose nav2_msgs/action/NavigateToPose \
  "{pose: {header: {frame_id: map}, position: {x: 2.0, y: 1.5}, orientation: {w: 1.0}}}"
```

## 动手步骤

1. 跑 M Ch8 的 Nav2 demo（TurtleBot3 或自己的机器人 + Gazebo）。
2. SLAM Toolbox 建图：遥控（`teleop_twist_keyboard`）把场地走一圈 → 存图。
3. 重启进导航模式，RViz 中点目标点 → 观察全局规划、局部控制、恢复行为（故意挡路看 spin/backup）。
4. 写一个 BT：`Sequence[ Condition(电量OK), Action(导航到A), Action(导航到B), Action(回窝) ]`，用 Groot2 跑通（M Ch7 + Ch8 的 BT Navigator 结合点）。

## 常见坑

- `frame_id` 必须 `map`，写 `odom`/`base_link` → 定位错乱或 action 直接失败。
- 建图时 LiDAR 数据率太低 / 场地太暗（反射弱）→ 图断裂、重影。
- 忘了 `/tf` 的 map→odom（定位节点没起）→ "localization failed"。
- lifecycle 节点没 activate → action server 不响应；`ros2 lifecycle get /bt_navigator` 检查。
- costmap 膨胀半径小于机器人半径 → 规划"穿过"障碍。
- BT 节点端口名拼错 → tick 直接失败（Groot2 里看 status 颜色）。

## 自测

1. Sequence 与 Fallback 的"短路"方向分别是什么？
2. BT 的 Blackboard 解决什么问题？
3. Nav2 里全局路径和局部路径分别由谁算？
4. 导航 action 失败时，Nav2 的"恢复"从哪来？
5. 建图（SLAM）与导航（localization）对 `/map` 的用法有何不同？

### 参考答案

1. Sequence：前一个失败→整体失败（停止尝试后面）；Fallback：前一个失败→尝试下一个，成功→停止。
2. 节点间共享数据（目标位姿、状态标志），避免把所有参数都写进 BT XML。
3. Planner Server（全局，A*/网格）与 Controller Server（局部，DWB/MPPI 类）。
4. BT Navigator 用 BT 定义恢复序列（spin→backup→wait→重定位），Behavior Server 提供 spin/backup 等原子行为。
5. 建图时 `/map` 是**输出**（SLAM 估计的地图）；导航时 `/map` 是**输入**（先验地图），机器人靠它定位并规划。

## 笔记清单（对照书本补写）

- [ ] 手绘 BT 节点类型表 + 每种一个机器人场景
- [ ] 抄写一个完整 BT XML 模板 + 注释
- [ ] 画 Nav2 架构图（7 个 server + lifecycle manager）
- [ ] 记录"建图→存图→导航"完整命令序列
- [ ] 整理导航失败恢复的 4 种行为及触发条件
