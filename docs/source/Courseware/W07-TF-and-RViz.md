# W07 · TF 与 RViz（课件）

> 对应阅读: S Ch10
> 里程碑: TF 树演示
> 配套笔记: 待写（建议命名 `Chapter-7-TF-and-RViz.md`）

## 学习目标

1. 说清 TF 树的结构与规则（单根、无环）；
2. 能手动广播 TF / static TF，并用 tf2 查询；
3. 在 RViz 中正确显示 RobotModel + TF。

## 核心概念

### 1. 坐标系与 TF 树

每个刚体（link）都有自己的坐标系；**TF 记录任意两坐标系之间的变换**（平移 + 旋转），且带时间戳。所有 TF 组成一棵树：

```
map            ← 全局地图坐标系（定位参考）
└── odom       ← 里程计坐标系（局部连续, 会漂移）
    └── base_link   ← 机器人本体
        ├── lidar_link
        ├── camera_link
        └── wheel_left_link / wheel_right_link
```

- **单根、无环**：A→B 和 B→A 只能有一个方向；成环 → tf2 直接报错。
- **static vs 动态**：固定不动的（link 之间）用 `StaticTransformBroadcaster`（广播一次即可）；随时间变的（map→odom、base→轮子）用 `TransformBroadcaster`（周期性）。

### 2. 为什么要"树"而不是一个大坐标系

机器人各部分**在不同时刻、以不同方式**运动；把每个 link 的坐标都表达成"相对父 link 的变换"，就能在任意时刻回答"传感器 A 相对地图在哪里"（= 链路上各段变换相乘）。

### 3. tf2 核心 API

| API | 用途 |
|---|---|
| `Buffer` | 存变换历史，支持按时间查询 |
| `TransformBroadcaster` | 广播动态 TF |
| `StaticTransformBroadcaster` | 广播静态 TF |
| `TransformListener` | 把 /tf 灌进 Buffer |
| `buffer.lookup_transform(target, source, time)` | 查 A→B 变换 |

### 4. RViz 显示要点

- **Fixed Frame** 必须选一个真实存在的 frame（通常 `base_link`）；
- RobotModel 显示需要两个话题：`robot_description`（URDF 文本）+ `/tf`；
- TF 显示会把整棵树画成坐标轴。

## 动手步骤

1. **手动广播**：写一个 50Hz 的节点，广播 `base_link → gripper`（绕 z 轴缓慢旋转 + 固定偏移）：

```python
from tf2_ros import TransformBroadcaster
from geometry_msgs.msg import TransformStamped

def tick(self):
    t = TransformStamped()
    t.header.stamp = self.get_clock().now().to_msg()
    t.header.frame_id = 'base_link'
    t.child_frame_id = 'gripper'
    t.transform.translation.x = 0.3
    t.transform.rotation.w = 1.0   # 先只给平移, 再练习欧拉角→四元数
    self.broadcaster.sendTransform(t)
```

2. **RViz 观察**：加 TF 显示，确认 `base_link`、`gripper` 都在树里。
3. **tf2 查询**：listener + buffer，查询 `gripper → base_link` 并打印位置。
4. **`view_frames`**：`ros2 run tf2_tools view_frames` 生成 `tmp_frames.pdf`，对照 S Ch10 检查自己的树。

## 常见坑

- **frame 名带前导 `/`** → tf2 里 frame 名一律**不加** `/`（话题才加）。
- 成环广播（A→B 与 B→A 都发）→ "received data out of order" / lookup 失败。
- static TF 用 100Hz 周期广播 → 刷屏且无必要；static 发 1–3 次即可。
- RViz 黑屏 / RobotModel 不显示 → Fixed Frame 选错、`robot_description` 话题没发、URDF 有问题。
- 时间戳为 0 或未来时间 → lookup_transform 报 "exceeds buffer"；检查 `header.stamp`。

## 自测

1. TF 树的两条铁律？
2. static TF 和动态 TF 的 broadcaster 有何不同？各举一个机器人上的例子。
3. 为什么 frame 名不能以 `/` 开头？
4. `lookup_transform('map','base_link')` 在内部做了什么？
5. 里程计 odom 坐标系为什么和 map 分开？

### 参考答案

1. 单根（一般 map 或 base_link 为根）；无环。
2. Static 用 `StaticTransformBroadcaster`（一次广播、永久有效）：link 间固定关系；动态用 `TransformBroadcaster`（周期广播）：map→odom、关节运动。
3. tf2 约定 frame 是纯名字（不带命名空间），带 `/` 会被当成不同 frame 导致查不到。
4. 沿树从 map 到 base_link 找到路径，把各段变换（含时间插值）连乘得到总变换。
5. odom 是"局部连续但会漂移"，map 是"全局一致但有跳变（重定位）"；分开后才能把漂移量（map→odom）与运动量（odom→base）解耦。

## 笔记清单（对照书本补写）

- [ ] 手绘差速机器人的完整 TF 树（含轮子）
- [ ] 抄写 tf2 五个核心 API + 一句话职责
- [ ] 记录 `view_frames` 输出树 + 标注 static/动态
- [ ] 用自己的话写"为什么需要 map/odom 两级"
- [ ] 整理 RViz 黑屏排查清单（≥4 条）
