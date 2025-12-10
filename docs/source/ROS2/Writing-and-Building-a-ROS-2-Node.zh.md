# 编写与构建 ROS 2 节点

**目录**
- [创建工作区](#创建工作区)
  - [构建工作区](#构建工作区)
- [创建包](#创建包)
  - [Python 包](#python-包)
  - [C++ 包](#c-包)
  - [构建包](#构建包)
- [创建 Python 节点](#创建-python-节点)
  - [构建节点](#构建-节点)
- [创建 C++ 节点](#创建-c-节点)
  - [构建并运行节点](#构建并运行-节点)
- [Python 与 C++ 节点模板](#节点模板)
- [检查与调试（Introspection）](#检查与调试)
  - [ros2 node 命令行](#ros2-node-命令行)
  - [运行时修改节点名称](#运行时修改节点名称)
- [总结](#总结)

## 创建工作区
当项目发展到需要多个应用或模块时，建议为每个应用或机器人创建独立的工作区（workspace），并以应用名或机器人名命名工作区目录。例如，机器人名称为 `ABC V3`，工作区可命名为 `abc_v3_ws`。

创建示例工作区：

```bash
$ cd ~
$ mkdir ros2_ws
$ cd ros2_ws/
$ mkdir src
```

### 构建工作区
在工作区根目录运行 `colcon build` 进行构建（`colcon` 是 ROS 2 的构建工具）。

```bash
$ cd ~/ros2_ws/
$ colcon build
```

构建后会生成三个目录：`build`（构建中间文件）、`install`（安装产物）和 `log`（构建日志）。每次完成构建后，需要 `source` 安装目录下的 `setup.bash`，以便环境能识别新安装的包：

```bash
$ source ~/ros2_ws/install/setup.bash
```

你可以将这条命令加入 `~/.bashrc`，以便每次打开终端自动生效：

```bash
source /opt/ros/<ros2-distro>/setup.bash
source ~/ros2_ws/install/setup.bash
```

## 创建包
每个节点都存在于一个包（package）中。因此要编写节点，先在工作区内创建包。Python 包与 C++ 包的结构不同。

### Python 包
使用 `ros2 pkg create` 创建最基本的 Python 包：

```bash
$ cd ~/ros2_ws/src/
$ ros2 pkg create my_py_pkg --build-type ament_python --dependencies rclpy
```

Python 包会包含 `my_py_pkg/` 源目录、`package.xml`、`setup.py` 等文件。实际的 Python 节点代码放在包内的模块目录（如 `my_py_pkg/`）中。

`setup.py` 中的 `entry_points` 用于注册可执行脚本，使得可以通过 `ros2 run <pkg> <executable>` 启动节点。

### C++ 包
创建 C++ 包的命令示例：

```bash
$ cd ~/ros2_ws/src/
$ ros2 pkg create my_cpp_pkg --build-type ament_cmake --dependencies rclcpp
```

C++ 包包含 `CMakeLists.txt`、`include/`、`src/`、`package.xml` 等。源码一般放在 `src/`，头文件放在 `include/`。

### 构建包
回到工作区根目录运行 `colcon build`，可以构建所有包；也可以使用 `--packages-select` 指定单个包构建：

```bash
$ colcon build --packages-select my_py_pkg
# 或
$ colcon build --packages-select my_cpp_pkg
```

构建完成后别忘了 `source ~/ros2_ws/install/setup.bash`。

## 创建 Python 节点
创建 Python 节点的步骤：
1. 在包的源目录下创建 Python 文件并设为可执行。
2. 编写继承自 `rclpy.node.Node` 的类，定义计时器、发布者、订阅者等。
3. 在 `main()` 中初始化 `rclpy`、创建节点并 `rclpy.spin()`。

示例（简化）：
```python
#!/usr/bin/env python3
import rclpy
from rclpy.node import Node

class MyCustomNode(Node):
    def __init__(self):
        super().__init__('my_node_name')
        self.counter_ = 0
        self.timer_ = self.create_timer(1.0, self.print_hello)

    def print_hello(self):
        self.get_logger().info("Hello %d" % self.counter_)
        self.counter_ += 1

def main(args=None):
    rclpy.init(args=args)
    node = MyCustomNode()
    rclpy.spin(node)
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

构建并安装时，在 `setup.py` 的 `entry_points` 下添加类似：

```python
entry_points={
    'console_scripts': [
        'test_node = my_py_pkg.my_first_node:main'
    ],
},
```

建议使用 `--symlink-install`（只对 Python 有效）以便在开发时修改代码无需每次重建：

```bash
$ colcon build --packages-select my_py_pkg --symlink-install
```

## 创建 C++ 节点
C++ 节点示例（类继承自 `rclcpp::Node`），在构造函数中创建计时器或发布/订阅对象：

```cpp
#include "rclcpp/rclcpp.hpp"

class MyCustomNode : public rclcpp::Node
{
public:
    MyCustomNode() : Node("my_node_name"), counter_(0)
    {
        timer_ = this->create_wall_timer(std::chrono::seconds(1),
                                         std::bind(&MyCustomNode::print_hello, this));
    }

    void print_hello()
    {
        RCLCPP_INFO(this->get_logger(), "Hello %d", counter_);
        counter_++;
    }

private:
    int counter_;
    rclcpp::TimerBase::SharedPtr timer_;
};
```

在 `CMakeLists.txt` 中添加可执行文件并链接依赖：

```cmake
find_package(rclcpp REQUIRED)
add_executable(test_node src/my_first_node.cpp)
ament_target_dependencies(test_node rclcpp)
install(TARGETS
  test_node
  DESTINATION lib/${PROJECT_NAME}
)
```

构建后记得 `source` 工作区并运行：

```bash
$ colcon build --packages-select my_cpp_pkg
$ source ~/ros2_ws/install/setup.bash
$ ros2 run my_cpp_pkg test_node
```

## 节点模板（Python & C++）
文件中提供了简洁的 Python 和 C++ 节点模板，用户可按需修改类名、节点名与回调内容。

## 检查与调试（Introspection）
使用 `ros2 node`、`ros2 topic`、`ros2 service` 等命令行工具可以查看当前系统中的节点、主题、服务与参数信息：

```bash
$ ros2 node list
$ ros2 node info /my_node_name
```

### 运行时修改节点名称
可通过 `--ros-args -r __node:=new_node_name` 在运行时重映射节点名称：

```bash
$ ros2 run my_py_pkg my_first_node.py --ros-args -r __node:=new_node_name
```

## 总结
- 创建与构建 ROS 2 工作区和包（Python / C++）。
- 在包中编写节点代码并通过 `colcon build` 构建。
- 使用 `entry_points` 或 `CMakeLists.txt` 注册可执行项以便用 `ros2 run` 启动。

（翻译完成）
