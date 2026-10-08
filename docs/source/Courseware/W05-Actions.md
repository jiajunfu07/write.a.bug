# W05 · Actions：当 Service 不够用（课件）

> 对应阅读: S Ch7
> 里程碑: action 反馈 / 取消演示
> 配套笔记: 待写（建议命名 `Chapter-4-Actions-When-Services-Are-Not-Enough.md`）

## 学习目标

1. 说清 action 解决的三类问题：长耗时、进度反馈、可取消；
2. 掌握 action 文件的 goal/result/feedback 三段结构；
3. 写出最小可用的 action server + client（含取消）。

## 核心概念

### 1. 为什么需要 action

Service 假设"一问一答很快结束"。但导航 30 秒、机械臂抓取 5 秒且需要"还剩多少"的进度、用户可能中途喊停 —— service 表达不了这三点。Action = **service 的升级版**：

```
Client ──goal──▶ Server      (一次性, 同 service 请求)
Client ◀─feedback── Server   (流式, 同 topic, 可多次)
Client ◀─result── Server     (一次性, 成功/失败 + 结果)
Client ──cancel──▶ Server    (可选)
```

### 2. action 文件

```
# goal
int32 order
---
# result
int32 sequence[]
---
# feedback
int32 partial_sum
```

三段：goal（要做什么）、result（最终给了什么）、feedback（过程播报）。

### 3. 官方示例先跑起来（强烈建议第一步）

```bash
sudo apt install ros-jazzy-demo-nodes-py ros-jazzy-fibonacci-msgs -y
ros2 run demo_nodes_py fibonacci_action_server
# 另开终端
ros2 action list
ros2 action send_goal /compute_demo_node fibonacci_msgs/action/Fibonacci "{order: 10}"
ros2 action echo /compute_demo_node   # 看 feedback 流
```

### 4. Server 骨架（Python，以 Fibonacci 为例）

```python
from rclpy.action import ActionServer
from my_act.action import Fibonacci

def handle_goal(self, goal_request, goal_handle):
    if goal_request.order < 0:
        return GoalResponse.REJECT
    return GoalResponse.ACCEPT

def handle_accepted(self, goal_handle):
    self.get_logger().info('goal accepted')

def handle_cancel(self, goal_handle):
    if goal_handle.is_cancel_requested:
        self.get_logger().info('cancelling...')
        return CancelResponse.ACCEPT
    return CancelResponse.REJECT

def handle_task(self, goal_handle):
    for i in range(1, goal_handle.request.order + 1):
        if goal_handle.is_cancel_requested:
            goal_handle.canceled()
            return Fibonacci.Result()
        # ... 计算 ...
        goal_handle.publish_feedback(Fibonacci.Feedback(partial_sum=...))
    result = Fibonacci.Result(sequence=[...])
    goal_handle.succeed()
    return result
```

### 5. Client 骨架

```python
from rclpy.action import ActionClient, GoalStatus
self.ac = ActionClient(self, Fibonacci, '/compute_demo_node')
goal = Fibonacci.Goal(order=10)
self.send_future = self.ac.send_goal_async(goal, feedback_handler=self.fb)
rclpy.spin_until_future_complete(self, self.send_future)
self.result_future = self.send_future.result().goal_handle.get_result_async()
rclpy.spin_until_future_complete(self, self.result_future)
print(self.result_future.result().result.sequence)
```

## 动手步骤

1. 跑通官方 Fibonacci（上面 3 条命令），`ros2 action echo` 观察 feedback。
2. **写自己的 action**（更贴近机器人）：`Navigate` —— goal(x, y)；feedback(current_x, current_y, distance_left)；result(distance_traveled, success)。用 1Hz 的"假运动"逐步逼近目标点即可。
3. **取消实验**：goal 发出 1 秒后，从另一个终端 `ros2 action cancel /navigate ...`（或 client 端 `goal_handle.cancel_goal()`），验证 server 正确收尾。
4. 画一张 goal/feedback/result/cancel 四通道时序图（放进笔记）。

## 常见坑

- server 端忘了 `rclpy.spin(node)`（ActionServer 挂在 node 上，不 spin 不干活）。
- 任务循环里不检查 `is_cancel_requested` → 取消无效。
- 结束时没调 `goal_handle.succeed()`/`canceled()`/`abort()` → client 一直等 result。
- Python client 里 `send_goal_async` 的 future 没保存引用 → 被 GC，结果丢。
- action 名与 `ros2 action list` 看到的要一致；`send_goal` 第二个参数必须 `pkg/action/Name`。

## 自测

1. action 相比 service 多了哪两个能力？
2. action 文件的三段分别对应什么时机？
3. `handle_goal` 返回 REJECT 和任务中 `abort()` 有什么区别？
4. client 取消 goal 后，server 端靠什么感知？
5. 给"机械臂抓取"设计一个 action（写出 goal/result/feedback 字段）。

### 参考答案

1. 中间进度反馈（feedback 流）+ 可取消（cancel 通道）。
2. goal=请求时、feedback=执行中（可多次）、result=完成时（一次）。
3. REJECT 是"根本不做"（还没开始）；abort 是"做到一半放弃"，需要自己触发。
4. 在任务循环里轮询 `goal_handle.is_cancel_requested`（server 必须主动检查）。
5. 例：goal(target_pose, grasp_width)；feedback(joint_positions, gripper_state)；result(success, final_pose, force)。

## 笔记清单（对照书本补写）

- [ ] 画 action 四通道时序图（client ↔ server）
- [ ] 抄写 action 文件语法 + 一个自己设计的 action 定义
- [ ] 记录官方 Fibonacci 的完整运行日志（goal→feedback×N→result）
- [ ] 用自己的话总结 service vs action 选型标准
- [ ] 记录一次取消实验的日志 + 结论
