# 在 Docker 容器中运行 ROS 2 节点

本页来自社区贡献。说明如何在 Docker 容器中运行 ROS 2 节点——可以在同一个容器内运行两个节点，也可以在两个独立容器中分别运行。

## 在单个 Docker 容器中运行两个节点

拉取 ROS 官方镜像（将 `{DISTRO}` 替换为你的 ROS 2 发行版，例如 `humble`、`iron` 等）：

```bash
$ docker pull osrf/ros:{DISTRO}-desktop
```

交互式运行镜像：

```bash
$ docker run -it osrf/ros:{DISTRO}-desktop
```

容器内可以使用 `ros2` 命令行工具，例如：

```bash
$ ros2 --help
$ ros2 pkg list
$ ros2 pkg executables <pkg_name>
```

运行一个最小示例：在容器中启动 `demo_nodes_cpp` 包中的 `listener`（订阅者）和 `talker`（发布者）：

```bash
$ ros2 run demo_nodes_cpp listener &
$ ros2 run demo_nodes_cpp talker
```

## 在两个独立的 Docker 容器中运行两个节点

在第一个终端运行发布者：

```bash
$ docker run -it --rm osrf/ros:{DISTRO}-desktop ros2 run demo_nodes_cpp talker
```

在第二个终端运行订阅者：

```bash
$ docker run -it --rm osrf/ros:{DISTRO}-desktop ros2 run demo_nodes_cpp listener
```

你也可以使用 `docker-compose`（示例使用 `version: '2'`）来同时启动两个容器，示例 `docker-compose.yml`：

```yaml
version: '2'

services:
  talker:
    image: osrf/ros:{DISTRO}-desktop
    command: ros2 run demo_nodes_cpp talker
  listener:
    image: osrf/ros:{DISTRO}-desktop
    command: ros2 run demo_nodes_cpp listener
    depends_on:
      - talker
```

运行 `docker compose up` 即可启动服务，用 `Ctrl+C` 停止并退出。

（翻译完成）
