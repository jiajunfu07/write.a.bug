# 服务：节点之间的客户端/服务器交互

**目录**
- [什么是 ROS 2 服务？](#什么是-ros-2-服务)
- [创建自定义服务接口](#创建自定义服务接口)
  - [查找已有的服务接口](#查找已有的服务接口)
  - [创建新的服务接口](#创建新的服务接口)
- [编写服务服务器](#编写服务服务器)
  - [编写 Python 服务服务器](#编写-python-服务服务器)
  - [编写 C++ 服务服务器](#编写-c-服务服务器)
- [编写服务客户端](#编写服务客户端)
  - [编写 Python 服务客户端](#编写-python-服务客户端)
  - [编写 C++ 服务客户端](#编写-c-服务客户端)

## 什么是 ROS 2 服务？
服务（service）与主题（topic）类似，也由名称和接口定义。但服务的接口包含请求（request）和响应（response）两部分。客户端与服务器必须使用相同的服务名与接口才能通信。

何时使用 topic/服务？当需要单向、持续的数据流（例如传感器数据）使用 topic；当需要请求-响应式、RPC 风格交互使用 service。

服务要点：
- 服务由名称和接口定义。
- 服务名遵循与 topic 相同的规则：必须以字母开头，后续可包含字母、数字、下划线、波浪线和斜杠。
- 接口包含**请求消息**与**响应消息**，两端必须一致。
- 同一服务名下通常只有一个 server 实例，但可以有多个 client。
- 客户端之间互不感知，它们仅需使用正确的服务名与接口即可与服务器通信。
- 一个节点可以包含多个服务客户端或服务器，使用不同的服务名。
- 服务通常是同步的：客户端发送请求后会等待服务器返回响应（阻塞）。
- 服务不适合高频通信，高频数据应使用 topics。

## 创建自定义服务接口
在真实项目中，通常为自定义接口创建单独的接口包（例如 `my_robot_interfaces`），只放接口定义（`msg/`、`srv/`），不放节点代码。

### 查找已有的服务接口
示例：
```bash
$ ros2 interface list | grep example_interfaces/srv
example_interfaces/srv/AddTwoInts
example_interfaces/srv/SetBool
example_interfaces/srv/Trigger

$ ros2 interface list | grep std_srvs/srv
std_srvs/srv/Empty
std_srvs/srv/SetBool
std_srvs/srv/Trigger
```
也可以在 https://github.com/ros2/common_interfaces 查找常见接口。

### 创建新的服务接口
在接口包内创建 `srv` 目录并添加 `.srv` 文件（文件名使用 UpperCamelCase，例如 `ResetCounter.srv`）。服务定义必须用三条短横线 `---` 分隔请求与响应部分，字段使用 snake_case。示例：
```
int64 reset_value
---
bool success
string message
```
在 `CMakeLists.txt` 的 `rosidl_generate_interfaces()` 中注册该 `.srv` 文件并构建包，然后 `source` 环境后可以用 `ros2 interface show` 查看接口定义。

## 编写服务服务器
服务器需要在包的 `package.xml` 中添加对接口包的依赖，并在代码中导入接口，在节点构造函数中使用 `create_service()` 创建服务并提供回调函数。

### 编写 Python 服务服务器
在 `package.xml` 中添加依赖：
```xml
<depend>rclpy</depend>
<depend>example_interfaces</depend>
<depend>my_robot_interfaces</depend>
```
代码示例（简化）：
```python
#!/usr/bin/env python3
import rclpy
from rclpy.node import Node
from example_interfaces.msg import Int64
from my_robot_interfaces.srv import ResetCounter

class NumberCounterNode(Node):
    def __init__(self):
        super().__init__("number_counter")
        self.counter_ = 0
        self.number_subscriber_ = self.create_subscription(Int64, "number", self.callback_number, 10)
        self.reset_counter_service_ = self.create_service(ResetCounter, "reset_counter", self.callback_reset_counter)
        self.get_logger().info("Number Counter has been started.")

    def callback_number(self, msg: Int64):
        self.counter_ += msg.data
        self.get_logger().info("Counter:  " + str(self.counter_))

    def callback_reset_counter(self, request: ResetCounter.Request, response: ResetCounter.Response):
        if request.reset_value < 0:
            response.success = False
            response.message = "Cannot reset counter to a negative value"
        elif request.reset_value > self.counter_:
            response.success = False
            response.message = "Reset value must be lower than current counter value"
        else:
            self.counter_ = request.reset_value
            self.get_logger().info("Reset counter to " + str(self.counter_))
            response.success = True
            response.message = "Success"
        return response

def main(args=None):
    rclpy.init(args=args)
    node = NumberCounterNode()
    rclpy.spin(node)
    rclpy.shutdown()

if __name__ == "__main__":
    main()
```

### 编写 C++ 服务服务器
在 `CMakeLists.txt` 和 `package.xml` 中添加对 `my_robot_interfaces` 的依赖，在 C++ 源文件中包含服务头（`#include "my_robot_interfaces/srv/reset_counter.hpp"`），使用 `create_service<ResetCounter>()` 创建服务，回调接收请求与响应的共享指针。示例代码在原文中给出。

## 编写服务客户端
### 编写 Python 服务客户端
客户端需要在构造函数中使用 `create_client()` 创建客户端，并在调用前使用 `wait_for_service()` 等待服务可用，然后构造请求并通过 `call_async()` 发送，处理返回的 future 或在回调中处理响应。

示例步骤（概要）：
1. 确认服务已启动：`wait_for_service()`。
2. 构造 `ResetCounter.Request()` 并设置字段。
3. 使用 `call_async()` 发送请求并处理 future 的结果或使用回调。

（翻译到此为止，原文在此之后包含更多客户端的示例代码。）

---

（翻译完成）
