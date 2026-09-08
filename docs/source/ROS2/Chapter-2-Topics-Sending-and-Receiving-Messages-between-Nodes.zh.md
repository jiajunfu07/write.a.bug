# 第2章：主题：节点之间发送与接收消息

**目录**
- [第2章：主题：节点之间发送与接收消息](#第2章主题节点之间发送与接收消息)
  - [什么是 ROS 2 主题？](#什么是-ros-2-主题)
  - [编写发布者节点](#编写发布者节点)
    - [编写 Python 发布者](#编写-python-发布者)
    - [编写 C++ 发布者](#编写-c-发布者)
  - [编写订阅者节点](#编写订阅者节点)
    - [编写 Python 订阅者](#编写-python-订阅者)
    - [编写 C++ 订阅者](#编写-c-订阅者)
  - [处理主题的附加工具](#处理主题的附加工具)
    - [`ros2 topic` 命令行工具](#ros2-topic-命令行工具)
  - [创建自定义接口（重要）](#创建自定义接口重要)

## 什么是 ROS 2 主题？
主题由名称和接口定义。

要点：
- 主题由**名称**和**接口（消息类型）**定义。
- 主题名称必须以字母开头，后续可以包含字母、数字、下划线和斜杠。
- 任何发布者或订阅者必须使用相同的接口（消息类型）。
- 发布者和订阅者是匿名的；它们并不知道彼此的存在，只知道自己在发布或订阅某个主题。
- 一个节点可以包含多个发布者和订阅者，甚至针对相同主题也可以如此。

## 编写发布者节点

### 编写 Python 发布者
在 `my_py_pkg` 包中创建 Python 文件并赋予可执行权限：

```bash
$ cd ~/ros2_ws/src/my_py_pkg/my_py_pkg
$ touch number_publisher.py
$ chmod +x number_publisher.py
```

使用节点模板并添加发布逻辑：

```python
#!/usr/bin/env python3
import rclpy
from rclpy.node import Node
from example_interfaces.msg import Int64

class NumberPublisher(Node):
    def __init__(self):
        super().__init__('number_publisher')
        self.number_ = 2
        self.number_publisher_ = self.create_publisher(Int64, 'number', 10)
        self.number_timer_ = self.create_timer(1.0, self.publish_number)
        self.get_logger().info('Number publisher node has been started.')

    def publish_number(self):
        msg = Int64()
        msg.data = self.number_
        self.number_publisher_.publish(msg)
```

说明：`create_publisher()` 的参数为（接口类型，主题名，队列深度）。当消息发布速度太快而订阅者跟不上时，队列会缓存消息以避免丢失。

注册并运行：在 `setup.py` 的 `entry_points` 中添加可执行项，然后从工作区根目录构建并运行：

```bash
$ cd ~/ros2_ws/
$ colcon build --packages-select my_py_pkg
$ source install/setup.bash
$ ros2 run my_py_pkg number_publisher
```

### 编写 C++ 发布者
在 `my_cpp_pkg` 的 `src` 目录中创建 `number_publisher.cpp`，并在文件头包含消息类型：

```c++
#include "rclcpp/rclcpp.hpp"
#include "example_interfaces/msg/int64.hpp"
```

在构造函数中创建发布者：

```c++
number_publisher_ = this->create_publisher<example_interfaces::msg::Int64>("number", 10);
```

发布者通常声明为类的私有成员：

```c++
rclcpp::Publisher<example_interfaces::msg::Int64>::SharedPtr number_publisher_;
```

构建并运行 C++ 节点（记得在 `CMakeLists.txt` 中添加依赖并安装目标）：

```bash
$ cd ~/ros2_ws/
$ colcon build --packages-select my_cpp_pkg
$ source install/setup.bash
$ ros2 run my_cpp_pkg number_publisher
```

## 编写订阅者节点

### 编写 Python 订阅者
创建 `number_subscriber.py` 并在构造函数中创建订阅者：

```bash
$ cd ~/ros2_ws/src/my_py_pkg/my_py_pkg
$ touch number_subscriber.py
$ chmod +x number_subscriber.py
```

```python
#!/usr/bin/env python3
import rclpy
from rclpy.node import Node
from example_interfaces.msg import Int64

class NumberSubscriber(Node):
    def __init__(self):
        super().__init__('number_counter')
        self.counter_ = 0
        self.number_subscription_ = self.create_subscription(
            Int64,
            'number',
            self.callback_number,
            10)
        self.get_logger().info('Number subscriber node has been started.')

    def callback_number(self, msg: Int64):
        self.counter_ += msg.data
        self.get_logger().info("Counter: " + str(self.counter_))
```

订阅者回调会接收消息对象作为参数（例如 `msg.data` 用于访问 Int64 的数据字段）。在 Python 中可以为回调参数注明类型以增强可读性和安全性。

### 编写 C++ 订阅者
在 C++ 中创建订阅者并绑定回调：

```c++
using namespace std::placeholders;

number_subscriber_ = this->create_subscription<example_interfaces::msg::Int64>(
    "number", 10, std::bind(&NumberCounterNode::callbackNumber, this, _1));
```

回调接收消息的共享指针并可通过 `->` 访问字段：`msg->data`。

## 处理主题的附加工具

### `ros2 topic` 命令行工具
列出主题：

```bash
$ ros2 topic list
/number
/param_events
/rosout
```

查看主题信息：

```bash
$ ros2 topic info /number
Type: example_interfaces/msg/Int64
Publisher count: 1
Subscriber count: 1
```

查看消息类型定义：

```bash
$ ros2 interface show example_interfaces/msg/Int64
int64 data
```

从命令行订阅并打印消息：

```bash
$ ros2 topic echo /number
data: 2
data: 2
...
```

从命令行发布消息（`-r` 指定频率 Hz）：

```bash
$ ros2 topic pub -r 2.0 /number example_interfaces/msg/Int64 "{data: 7}"
```

重映射主题名（运行时修改主题名）：

```bash
$ ros2 run my_py_pkg number_subscriber --ros-args -r number:=my_number
```

记录主题到 bag 文件：

```bash
$ mkdir ~/bags
$ cd ~/bags
$ ros2 bag record /number -o bag1
```

## 创建自定义接口（重要）
首先检查是否已有合适接口（如 `sensor_msgs/msg/Image`）。若无，则创建一个接口包（例如 `my_robot_interfaces`），在其中建立 `msg` 或 `srv` 目录并在 `CMakeLists.txt` 中使用 `rosidl_generate_interfaces()` 注册。

示例消息定义应使用内置类型或现有消息类型，字段使用 snake_case。例如：

```text
int64 version
float64 temperature
bool are_motors_ready
string debug_message
```

将自定义接口作为依赖添加到使用它的包的 `package.xml` 中，然后在 Python/C++ 中分别通过 `from my_robot_interfaces.msg import HardwareStatus` 或 `#include "my_robot_interfaces/msg/hardware_status.hpp"` 来使用。

---

（翻译完成）
