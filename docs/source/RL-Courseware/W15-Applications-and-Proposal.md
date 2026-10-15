# W15 · 应用案例 + 毕业开题（课件）

> 对应阅读: Ch16 Applications (PDF p.443–480) 重点 TD-Gammon/AlphaGo；Ch17 "Designing Reward Signals" (p.491–494)
> 里程碑: 开题报告（1 页）+ 评审
> 配套笔记: 建议 `Chapter-16-Applications.md`（待写）

## 学习目标

1. 把 TD-Gammon、AlphaGo(Zero)、Atari 三个案例映射回课程方法（表格→近似→搜索+学习）；
2. 理解 reward 设计的 4 个陷阱（reward hacking、稀疏、shortcut、credit assignment）；
3. 写出可执行、可验收、可评审的 1 页开题报告。

## 核心概念

### 1. TD-Gammon（16.1，"RL 第一次打赢人类"）

- 1990s，Tesauro：checkers，**无手工知识**（初始随机权重），自对弈 ~22 亿局；
- 方法 = 神经网络（backprop）+ TD 误差（W9/W12 的技术合体）；
- 意义：证明"数据 + 梯度 + 时间"可以胜过人类领域知识（AlphaGo 的直系祖先）。

### 2. Atari / DQN（16.5）

- Mnih 2013/2015：原始像素 → CNN 特征 → DQN（W10 的完整工业版）；
- 里程碑意义：**单一算法**玩 49 个游戏达到人类水平中位数 → 通用 agent 的可行性。

### 3. AlphaGo → AlphaGo Zero（16.6）

- AlphaGo：MCTS（W8）+ 两个网络（policy 选点、value 评估）+ 人类棋谱预训练；
- **AlphaGo Zero**：无人类棋谱，自对弈 + MCTS + 共享网络 → 20 小时超越 AlphaGo；
- 映射回课程：MCTS=Ch8、value 网络=W9/10、自对弈=无监督数据生成、搜索+学习混合=Ch8 的"planning at decision time"。

### 4. Reward 设计的 4 个陷阱（Ch17 p.491–494，开题必读）

| 陷阱 | 例子 | 缓解 |
|---|---|---|
| **reward hacking** | 视频游戏 agent"看穿墙"刷分 | 奖励关键行为而非最终分；审计策略 |
| **稀疏 reward** | 迷宫只有出口 +1 | 潜力基奖励 / 课程式 shaping / 逆 RL 概念 |
| **shortcut** | 导航 agent 走"捷径"绕过障碍 | 加约束项 / 多任务验证 |
| **credit assignment** | 100 步后失败，锅在哪步 | λ-return / advantage / 分阶段奖励 |

### 5. 三个案例的方法学脉络（写进笔记）

```
TD-Gammon (1995):  表格→线性/近似 + TD + 自对弈
DQN/Atari (2015):  近似 + off-policy + 工程稳定化 (replay/target)
AlphaGo Zero:      搜索(MCTS) + 学习(网络) + 自对弈, 无人类知识
```

→ 共同主题：**"搜索 + 学习 + 自生成数据"是 RL 从玩具到实战的三件套**。

## 动手步骤（开题周）

1. 通读 Ch16 三个案例（TD-Gammon / Atari / AlphaGo Zero），每个写 5 行"它用到了课程哪几周的工具"；
2. 读 Ch17 reward design 一节，列出"你毕业项目会踩的 2 个 reward 陷阱 + 预案"；
3. 按下方模板完成开题报告（存 `rl-course/docs/proposal.md`）；
4. 按 W16 课件的"评审检查清单"自评一遍，再找同学/mentor 过一遍。

## 开题报告模板（1 页）

1. **项目名 + 一句话目标**（能做什么、做到什么验收线）
2. **系统架构图**（环境/策略/价值/搜索/数据流，标注用哪章的方法）
3. **技术栈**（Gymnasium 环境 or 自建环境；算法：W2–W13 的哪些；库：SB3/torch 版本）
4. **里程碑 ×4**（各带验收标准：可运行命令 + 指标 + 截图）
5. **风险 ×3**（风险 / 概率 / 缓解；必须包含 1 个 reward 相关风险）
6. **rubric 自评目标分**（功能 35 / 实验 25 / 代码 20 / 文档答辩 20）
7. **引用**（R 书章节 + 参考论文 + 开源项目）

## 常见坑（开题）

- 目标写成"学会 AlphaGo" → 不可验收；改成"在 7×7 棋盘上 MCTS 击败 X 引擎，胜率 >Y%"。
- 没有"最小可演示版本" → W16 现场翻车；里程碑 1 必须是可跑的。
- reward 设计没写 → 评审必问；把 4 个陷阱对照表填进"风险"。
- 同时选 2 个方向（又做 MCTS 又做 DQN）→ 范围爆炸；二选一 + 1 个加分项。
- 数据/算力没估 → 训练超时；先跑 10 分钟 pilot 估吞吐。

## 自测

1. TD-Gammon 用了课程的哪三件工具？
2. AlphaGo Zero 相比 AlphaGo 去掉了什么？为什么还能更强？
3. 你的项目里 reward 怎么设计？最可能踩哪个陷阱？
4. "搜索 + 学习 + 自生成数据"三件套在你的项目里各占什么位置？
5. 你的里程碑 1 的验收命令是什么？（现场答）

### 参考答案（示例）

1. 函数近似（网络）+ TD 误差（W6/W12）+ 自对弈数据（无监督）。
2. 去掉人类棋谱与两个独立网络 → 单一目标（自对弈 + MCTS）让训练更自洽、目标函数一致，反而更强。
3. 例：迷宫导航用"−0.01/步 + 到达 +1 + 碰撞 −1"；最可能踩稀疏（大迷宫）→ 加路径长度 shaping 并审计 shortcut。
4. 搜索=决策时 MCTS/规划；学习=网络价值/策略；自生成=自对弈/仿真 rollout 产生训练数据。
5. 例：`python train.py --env maze-7x7 --iters 2000` 输出 "win_rate=0.72 > 0.60" 且附 3 局轨迹 gif。

## 笔记清单（对照书本补写）

- [ ] 三案例方法学脉络图（TD-Gammon→DQN→AlphaGo Zero）
- [ ] 每个案例 5 行"用到课程哪几周工具"
- [ ] reward 4 陷阱表 + 你项目的 2 个风险预案
- [ ] 完成 1 页开题报告（proposal.md）
- [ ] 记录 pilot run 的吞吐估算（steps/s、预计总时长）
