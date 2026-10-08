# W04 · Services + Parameters（课件）

> 对应阅读: S Ch6（services）、S Ch8（parameters）
> 里程碑: 运行时动态配置
> 配套笔记: `Chapter-3-Service-Client-Server-Interation-between-Nodes.md`、`Chapter-5-Parameters-Making-Nodes-More-Dynamic.md`

## 学习目标

1. 说清 service 的"同步请求-响应"语义，与 topic/action 的边界；
2. 定义自定义 srv 并实现 client/server；
3. 用 parameters 让同一份代码以不同配置运行，并用回调实现运行时热更新。

## 核心概念

### 1. 三种通信怎么选

| | topic | service | action |
|---|---|---|---|
| 语义 | 异步事件流 | 同步一问一答 | 长任务 + 反馈 + 可取消 |
| 关系 | 1→N | 1↔1 | 1↔1 |
| 应答 | 无 | 必有 | 最终必有 + 中间反馈 |
| 典型 | 传感器数据、命令 | 查询状态、开关 | 导航、抓取 |

### 2. srv 文件结构

```
# 请求
int32 x
int32 y
---
# 响应
bool success
string message
```

`---` 分隔 req/res。类型与 msg 语法一致。

### 3. server / client 骨架（Python）

```python
# server
from my_srv.srv import SetPoint
self.srv = self.create_service(SetPoint, 'set_point', self.handle_set)

def handle_set(self, request, response):
    self.get_logger().info(f'got target ({request.x}, {request.y})')
    response.success = True
    response.message = 'ok'
    return response

# client
self.cli = self.create_client(SetPoint, 'set_point')
while not self.cli.wait_for_service(timeout_sec=1.0):
    self.get_logger().info('waiting for service...')
future = self.cli.call_async(SetPoint.Request(x=10, y=20))
rclpy.spin_until_future_complete(self, future)
print(future.result())
```

### 4. Parameters = 节点级命名配置

- 与变量不同：参数有**名字 + 类型 + 生命周期**，可由外部在运行时读写。
- 三步：`declare_parameter` → `get_parameter` → （可选）`set_parameter` 触发回调。
- 常用 CLI：`ros2 param list /node`、`ros2 param get /node name`、`ros2 param set /node name value`。
- 启动时注入：`ros2 run pkg node --ros-args -p name:=value`。

### 5. 参数回调（动态化的关键，S Ch8 核心）

```python
self.add_on_set_parameters_callback(self.cb)

def cb(self, params):
    p = params[0]
    self.max_speed = p.value
    self.get_logger().info(f'config changed: max_speed={p.value}')
    return SetParametersResult(successful=True)
```

回调返回 `successful=True` 表示接受新值；返回 False 可**拒绝**修改（参数保持原值）。

## 动手步骤

1. **srv 实战**：定义 `SetPoint`（见上），server 打印收到的目标；client 节点每 2s 调一次；
   再用 CLI 验证：`ros2 service call /set_point my_srv/srv/SetPoint "{x: 10, y: 20}"`。
2. **参数节点**：声明 `robot_name`(string)、`max_speed`(double)、`enable_lidar`(bool)，启动时打印当前配置。
3. **热更新**：加上参数回调；运行中 `ros2 param set /robot max_speed 2.5`，观察日志行为变化。
4. **对比实验**：同一节点分别用 `--ros-args -p max_speed:=1.0` 和 `:=3.0` 启动，证明"一份代码、两种配置"。

## 常见坑

- client 不调 `wait_for_service()` 就 call → 偶发 "wait_for_service failed" / 无响应。
- `ros2 service call` 第二参必须带包名：`pkg/srv/Name`，不是裸名字。
- 没 `declare_parameter` 就 `get_parameter` → 得到 NULL，类型转换报错。
- 参数回调里忘了 `return SetParametersResult(...)` → 行为未定义/修改被静默拒绝。
- 把高频数据当参数发（比如 100Hz 的位姿）→ 参数不是流，用 topic。

## 自测

1. 什么任务该用 service 而不是 topic？举 2 个例子并说明原因。
2. srv 文件中 `---` 的作用？
3. `declare_parameter` 不给默认值会怎样？
4. 参数回调返回 `successful=False` 会发生什么？
5. "参数"和"topic"的本质区别是什么？

### 参考答案

1. 需要"应答/确认"或状态查询的短任务：开关激光雷达、请求当前里程计、切换模式。因为 topic 无应答，调用方不知道是否成功。
2. 分隔请求段与响应段。
3. 参数仍存在但值为 NULL；`get_parameter` 取出来是空值，后续使用时容易类型报错。
4. 修改被拒绝，参数保持旧值（可用来做范围校验）。
5. 参数 = 节点自身配置（低频、可写、有类型契约）；topic = 节点间数据流（高频、单向、不可寻址单个订阅者）。

## 笔记清单（对照书本补写）

- [ ] 抄写三种通信对比表 + 各配一个机器人场景
- [ ] 记录 srv 定义 + client/server 最小可运行代码
- [ ] 用自己的话写"参数回调为什么能实现热更新"
- [ ] 总结 `ros2 param` 三个命令 + 启动注入的用法
- [ ] 记录一次"修改被回调拒绝"的实验日志（加分）
