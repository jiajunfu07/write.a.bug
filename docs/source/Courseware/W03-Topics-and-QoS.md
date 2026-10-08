# W03 · Topics 与 QoS（课件）

> 对应阅读: S Ch5；M Ch3（topics 小节）
> 里程碑: 自定义消息 + QoS 演示
> 配套笔记: `Chapter-2-Topics-Sending-and-Receiving-Messages-between-Nodes.md`

## 学习目标

1. 说清 pub/sub 的异步特性与 1→N 扇出；
2. 能定义并使用自定义消息类型；
3. 掌握 4 个 QoS 维度，能解释"收不到数据"的 QoS 失配案例。

## 核心概念

### 1. 话题模型

- 发布者只管"喊话"，不知道有几个人听；订阅者各自独立收。
- **异步 + 无应答**：发了就完（与 service 的本质区别）。
- 消息类型强类型：`std_msgs/String`、`sensor_msgs/LaserScan`、`geometry_msgs/Twist`……用 `ros2 interface show <pkg/msg/Name>` 查看结构。

### 2. 自定义消息

```bash
ros2 interface create msg/SensorState int32 seq float32 temperature float32 humidity
```

`.msg` 语法：`类型 名称`，按行排列，顺序即内存顺序。Python 端 `from my_msgs.msg import SensorState`。

### 3. QoS 四维度（面试级考点）

| 维度 | 取值 | 含义 |
|---|---|---|
| reliability | RELIABLE / BEST_EFFORT | 丢不丢：可靠=保证送达；尽力=丢了不补 |
| durability | VOLATILE / TRANSIENT_LOCAL | 离线期消息：易失=不补；本地持久=补发最新 N 条 |
| history | KEEP_LAST / KEEP_ALL | 缓存策略：留最新 N 条 / 全留（受 depth 限） |
| depth | 整数 | 与 history 配套的缓存条数 |

**兼容性铁律**：发布者"承诺"不能超过订阅者"要求"。
- RELIABLE pub + BEST_EFFORT sub → 可以；
- **BEST_EFFORT pub + RELIABLE sub → 建不上连接**（订阅者要求保证，发布者给不出）—— 这是"收不到数据"的头号原因。

### 4. QoS 代码（Python）

```python
from rclpy.qos import QoSProfile, ReliabilityPolicy, HistoryPolicy
qos = QoSProfile(depth=1,
                 reliability=ReliabilityPolicy.TRANSIENT_LOCAL,
                 history=HistoryPolicy.KEEP_LAST)
self.pub = self.create_publisher(SensorState, '/sensor_state', qos)
```

C++ 等价：`rclcpp::QoS(1).transient_local().keep_last(1)`。

## 动手步骤

1. **基础 pub/sub**（`std_msgs/String`）：publisher 1Hz 发计数；subscriber 打印；`ros2 topic list/info/echo/hz/bw` 逐一体验。
2. **自定义消息**：在 W02 的包里加 `msg/SensorState.msg`（`ros2 interface create`），记得：
   - `package.xml` 加 `<depend>rosidl_default_generators</depend>` 并把 `msg` 文件传给 `rosidl_generate_interfaces`（Python 包在 setup.py）；
   - `colcon build` 后用 `ros2 interface show` 验证。
3. **QoS 实验**：
   - 发布者用 `TRANSIENT_LOCAL, depth=1`；
   - 启动发布者 5 秒后**再**启动订阅者 → 仍能立刻收到最后一条（durability 生效）；
   - 把发布者改回 `VOLATILE`，重复 → 订阅者什么都收不到（除非后续再发）。
4. **失配实验**：订阅者显式设 `ReliabilityPolicy.RELIABLE`，发布者为 `BEST_EFFORT` → 观察"一直收不到"，用 `ros2 topic info /sensor_state -v` 查看两端 QoS 验证。

## 常见坑

- 自定义 msg 构建失败 → `package.xml` 依赖 / setup.py 的 `rosidl_generate_interfaces` 没带上新文件。
- "明明发了收不到" → 90% 是 QoS 失配，用 `ros2 topic info <topic> -v` 对照两端。
- `ros2 topic echo` 是 BEST_EFFORT 订阅，会"掩盖"真实订阅者的失配问题。
- TRANSIENT_LOCAL 的 depth 设 0 或很大都会出问题：depth 必须是 ≥1 的有限值（KEEP_ALL 也受 max 限制）。
- 话题名不要带前导 `/` 混用：带 `/` 是全局名，不带是私有名 `~/...`，remap 行为不同。

## 自测

1. 列出 QoS 4 个维度及各自取值。
2. BEST_EFFORT 发布者 + RELIABLE 订阅者能否通信？为什么？
3. `TRANSIENT_LOCAL` 适合什么场景？举一个机器人里的例子。
4. pub/sub 与 client/server 的三个本质区别？
5. 为什么建议自定义消息类型而不是"JSON 字符串塞进 std_msgs/String"？

### 参考答案

1. reliability（RELIABLE/BEST_EFFORT）、durability（VOLATILE/TRANSIENT_LOCAL）、history（KEEP_LAST/KEEP_ALL）、depth。
2. 不能：订阅者要求"保证送达"，发布者只承诺"尽力而为"，协商失败。
3. 慢订阅者/后上线节点需要"当前状态"：如参数、标定、目标位姿。
4. 同步 vs 异步；1:1 应答 vs 1:N 广播；请求-响应语义 vs 事件流语义。
5. 类型安全、可内省（`ros2 interface show`）、工具链支持（echo/graph）、无序列化歧义。

## 笔记清单（对照书本补写）

- [ ] 画 pub/sub 数据流图（1 发布 → 3 订阅）
- [ ] 抄写 QoS 兼容矩阵（4×4 或书本中的表格）并加一句"我的理解"
- [ ] 记录 `ros2 topic info /... -v` 输出并标注每行含义
- [ ] 总结"收不到数据"的 5 步排查法
