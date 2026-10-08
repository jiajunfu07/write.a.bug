# W02 · 第一个节点：Python 与 C++（课件）

> 对应阅读: S Ch3（核心概念）、S Ch4（编写节点）；M Ch2 (p.31–72)、M Ch3 (p.75–117)
> 里程碑: P0 greeting 节点 · 交付: repo + 2 个节点 + README
> 配套笔记: `Chapter-1-Writing-and-Building-a-ROS-2-Node.md`、`Chapter-0-...docker...md`

## 学习目标

1. 分清 workspace / package / node 三个层级；
2. 用 `colcon` 从零创建、构建、运行一个 Python 节点和一个 C++ 节点；
3. 看懂 `setup.py entry_points` 如何把命令映射到代码；
4. 用 `ros2 node`、`rqt_graph` 观察自己的节点。

## 核心概念

### 1. 三个层级

```
workspace (hello_ws)
└── src/                  # 你写的包都放这里
    ├── hello_ros2/       # package: 构建/安装/依赖的最小单元
    │   ├── package.xml   # 元数据 + 依赖
    │   ├── setup.py      # Python 包: 注册 entry_points
    │   └── src/ 或 包名目录/  # 代码
    └── cpp_greeter/      # 另一个包 (C++)
build/  install/  log/    # colcon 生成, 勿手改
```

- **node** = 一个"计算 + 通信"单元，通常是一个进程；一个包可以含多个节点。
- **colcon** 只做"把 src/ 里所有包按 ament 规则构建到 install/"，Python 包和 C++ 包规则不同（ament_python / ament_cmake）。

### 2. Python 节点骨架（背下来）

```python
import rclpy
from rclpy.node import Node
from std_msgs.msg import String

class GreetingPublisher(Node):
    def __init__(self):
        super().__init__('greeting_publisher')      # 节点名
        self.timer = self.create_timer(1.0, self.tick)
        self.count = 0
        self.get_logger().info('greeting_publisher started')

    def tick(self):
        self.count += 1
        self.get_logger().info(f'hello ros2, #{self.count}')

def main(args=None):
    rclpy.init(args=args)
    node = GreetingPublisher()
    rclpy.spin(node)          # 事件循环: 触发 timer/回调
    node.destroy_node()
    rclpy.shutdown()

if __name__ == '__main__':
    main()
```

### 3. C++ 节点骨架

```cpp
#include "rclcpp/rclcpp.hpp"
#include <memory>

class Greeter : public rclcpp::Node {
public:
  Greeter() : Node("greeter"), count_(0) {
    timer_ = this->create_wall_timer(std::chrono::seconds(1), [this]() {
      RCLCPP_INFO(this->get_logger(), "hello from C++ #%d", count_++);
    });
  }
private:
  int count_;
  rclcpp::TimerBase::SharedPtr timer_;
};

int main(int argc, char ** argv) {
  rclcpp::init(argc, argv);
  auto node = std::make_shared<Greeter>();
  rclcpp::spin(node);
  rclcpp::shutdown();
  return 0;
}
```

### 4. entry_points 是"命令 → 代码"的胶水

`setup.py` 里声明后，`ros2 run pkg 命令名` 才知道该 import 哪个模块、调哪个 `main()`：

```python
entry_points={
    'console_scripts': [
        'greeting_publisher = hello_ros2.greeting_publisher:main',
    ],
},
```

## 动手步骤

1. **Python 包**：

```bash
mkdir -p hello_ws/src && cd hello_ws
ros2 pkg create --build-type ament_python --license apache-2.0 hello_ros2
# 写入 greeting_publisher.py, 并把 entry_points 加进 setup.py,
# package.xml 里补 <depend>rclpy</depend> <exec_depend>std_msgs</exec_depend>
colcon build
source install/setup.bash
ros2 run hello_ros2 greeting_publisher
```

2. **C++ 包**：

```bash
ros2 pkg create --build-type ament_cmake cpp_greeter
# 把 greeter_node.cpp 放进 src/, 修改 CMakeLists.txt:
#   find_package(std_msgs REQUIRED)
#   add_executable(greeter src/greeter_node.cpp)
#   target_link_libraries(greeter rclcpp::rclcpp)
#   install(TARGETS greeter DESTINATION lib/${PROJECT_NAME})
colcon build && source install/setup.bash
ros2 run cpp_greeter greeter
```

3. **观察**：`ros2 node list`、`ros2 node info /greeting_publisher`、`ros2 run rqt_graph rqt_graph`。
4. **选做（对应本站 Chapter-0）**：把两个节点分别装进两个 Docker 容器，验证跨容器通信（同一 `ROS_DOMAIN_ID`）。

## 常见坑

- `colcon build` 成功但 `ros2 run` 找不到 → 忘了 `source install/setup.bash`。
- `entry_points` 路径写错 → `ros2 run` 抛 `ModuleNotFoundError`；对照"模块名.文件:函数"格式检查。
- C++ 编译报 `std_msgs not found` → CMakeLists 缺 `find_package(std_msgs REQUIRED)` + `target_link_libraries`。
- 两个终端一个能跑一个不能 → 另一个没 source（或 `ROS_DOMAIN_ID` 不同）。
- Python 缩进/`super().__init__` 漏掉 → 节点名冲突或崩溃，先看 traceback。

## 自测

1. workspace、package、node 三者的关系？一个包能有几个节点？
2. `entry_points` 的作用？删掉它 `ros2 run` 会怎样？
3. `ros2 run` 和 `ros2 launch` 的区别？
4. Python `create_timer` 和 C++ `create_wall_timer` 等价吗？`rclpy.spin` 对应 C++ 的什么？
5. `colcon build` 失败时，按什么顺序排查？

### 参考答案

1. workspace 是容器（src/ + build/install/log）；package 是构建与依赖的最小单元；node 是运行时的进程级单元。一个包可含多个节点（entry_points 里注册多条）。
2. 声明"命令名 = 模块:main"的映射；删掉后 `ros2 run` 报找不到可执行入口。
3. `ros2 run` 跑单个节点；`launch` 一次性启动一组节点 + 参数/remap。
4. 等价（都是周期性回调）；`rclpy.spin` 对应 `rclcpp::spin`（事件循环）。
5. 看 log/ 里最后报错的包 → package.xml 依赖 → setup.py/CMakeLists → 代码语法；`colcon build --packages-select <pkg> --symlink-install` 可加速定位。

## 笔记清单（对照书本补写）

- [ ] 画 workspace/package/node 层级图（S Ch4）
- [ ] 记录 `ros2 pkg create` 生成的目录结构 + 每个文件职责
- [ ] 抄写并解释 entry_points 三要素（命令、模块、函数）
- [ ] 记录 Python 与 C++ 骨架代码的差异点（至少 3 处）
- [ ] 选做：记录跨容器通信的 3 个关键变量/条件
