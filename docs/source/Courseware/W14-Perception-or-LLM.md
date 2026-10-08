# W14 · 感知 / LLM 选修（课件）

> 对应阅读: M Ch10（感知）、M Ch14 (p.443 起, LLM)
> 里程碑: 感知 或 LLM 演示（二选一）
> 配套笔记: 待写（建议命名 `Chapter-14-Perception.md` / `Chapter-15-LLM-Agent.md`）

## 学习目标（两路线通用）

1. 把传感器数据（图像/点云/语言指令）接入 ROS 2 决策层；
2. 完成一个可演示的闭环；
3. 理解"感知/大模型输出不可靠"时的工程对策。

## 路线 A · 感知栈（M Ch10）

### 核心概念

- **相机管线三件套**：`/camera/image_raw`（`sensor_msgs/Image`）+ `/camera/camera_info`（内参/畸变）+ （可选）`/camera/image_rect`。
- **相机坐标系**：x 右、y 下、z 前（光轴）——与 base_link 的 z 上不同，TF 里要转对。
- **点云**：`sensor_msgs/PointCloud2`；常用操作：体素下采样（降量）、欧氏聚类（找物体）。
- **cv_bridge**：`sensor_msgs/Image` ↔ OpenCV `Mat`，做检测（Haar/DNN）。
- 时间对齐：相机帧与 TF/odom 的时间戳（`use_sim_time`）。

### 动手

1. Gazebo 中给相机模型加 sensor 插件 → RViz 显示 `/camera/image_raw`。
2. 写节点：cv_bridge 转 Mat → 简单检测（颜色阈值或 Haar 人脸）→ 发布 `geometry_msgs/PoseStamped`（目标在 base_link 下的位置）。
3. （加分）把检测结果作为 Nav2/MoveIt 的输入（"看见目标 → 过去"）。

### 常见坑

- 忘了 `/camera_info` 就解投影 → 像素↔3D 映射错位。
- 相机坐标系 y 轴向下，直接当 base_link 用 → 目标位置上下颠倒。
- 压缩图像（`CompressedImage`）与原始 Image 混用 → 桥接报格式错。

## 路线 B · LLM Agent（M Ch14）

### 核心概念

- **Agent 三要素**：LLM（大脑）+ Tools（手脚，映射到 ROS action/service）+ 记忆/状态。
- **ROS 2 里的形态**：一个普通节点，订阅/发布照常，只是"决策"外包给大模型 API。
- **Tool calling**：把 `NavigateToPose` 之类的 action 描述成 JSON schema 给 LLM，LLM 返回结构化调用参数。
- **安全层**：白名单工具、参数范围校验、危险动作二次确认、限流与超时。
- M Ch14 示例：LLM 控制 turtlesim（Spawn / SetPen / 转向），OpenAI API key 走参数注入。

### 动手

1. 按 M Ch14 搭 agent 节点（节点 + API 客户端 + tool 注册）。
2. 自然语言指令 → action 调用："把小海龟移到 (2,1)" → `actionlib` 发 goal。
3. （加分）多轮对话 + 简单记忆（会话上下文存 blackboard/参数）。

### 常见坑

- 把 API key 硬编码进代码/仓库 → 用参数 + 环境变量，`.gitignore` 住。
- LLM 幻觉参数（x=9999）→ 必须在工具执行前做范围校验。
- 同步阻塞等 API → 节点卡死；用异步/独立 executor。
- 没有限流 → 连续刷屏调用，费用和延迟爆炸。

## 自测（按所选路线作答）

1. A: `camera_info` 里有什么？缺了它会怎样？
2. A: 相机坐标系与 base_link 的关键差异？
3. B: tool calling 的"契约"由谁定义？格式是什么？
4. B: LLM 返回了一个危险动作（全速前进），你的节点该怎么设计拦截？
5. 两路线通用：感知/LLM 输出"不可靠"时，机器人系统的兜底策略有哪些？（≥3 条）

### 参考答案

1. 内参（fx, fy, cx, cy）、畸变系数、图像尺寸与编码；缺了它无法做像素→3D 解投影，检测结果会错位。
2. 光轴：相机 z 向前、y 向下；base_link 一般 z 向上 —— 需要 TF 旋转，直接用会导致符号/轴序错误。
3. 由开发者（ROS 侧）定义，JSON schema（名称、参数、类型、范围）；LLM 只负责按 schema 填参数。
4. 白名单 + 参数边界校验 + 速度上限 + 需人工确认的确认话题/按钮 + 可全局急停（bypass LLM）。
5. 边界校验、超时与重试、降级策略（回退到规则控制器）、人确认回路、行为日志与回放。

## 笔记清单

- [ ] 路线 A：抄写 camera 三件套话题 + 坐标轴图
- [ ] 路线 A：记录 cv_bridge 最小代码 + 一次检测结果的 pose 输出
- [ ] 路线 B：画 agent 架构图（LLM ↔ 节点 ↔ ROS action）
- [ ] 路线 B：抄写一个 tool 的 JSON schema + 对应的 ROS action 映射
- [ ] 通用：整理"不可靠输出兜底清单"并写进你的项目（若选 C/D 毕业项目必用）
