# Reinforcement Learning — 课程大纲与学习计划

> **课程编号**: RL-201 · **学分**: 3 · **周期**: 16 周（每周 2 讲时 + 4 实验时，共 96 学时）
> **面向人群**: 计算机 / 自动化 / 数学专业本科生及以上，以及系统自学者
> **教材**:
> - 主教材 **R** — *Reinforcement Learning: An Introduction*, 2nd ed. (draft, 2020, 548 页), Richard S. Sutton & Andrew G. Barto
> - 辅助材料: OpenAI Spinning Up、stable-baselines3 文档、Gymnasium 文档

## 1. 课程概述

本书是强化学习领域的"圣经"，结构清晰：**Part I 表格法**（Bandit → MDP → DP → MC → TD → n-step → 规划/Dyna）→ **Part II 近似法**（函数近似 → DQN/半梯度 → off-policy 发散 → 资格迹 → 策略梯度）→ **Part III 深入**（心理学、神经科学、应用案例、前沿）。

课程按"理论一周 + 实验一周一个算法"的节奏组织，每 4–6 周一个里程碑项目，最终交付一个完整 RL 项目（含代码、曲线、报告与答辩）。

### 学习产出（学完你能够）

1. 用 2–3 句话说清 RL 的三要素（agent、environment、reward）与 reward hypothesis；
2. 手写并调试 DP / MC / TD(0) / Sarsa / Q-learning / n-step / Dyna 七套表格算法，并在同一任务上对比学习曲线；
3. 理解 Bellman 方程、GPI、on/off-policy、bootstrapping 与 sampling 的完整谱系；
4. 用线性近似与神经网络（tile coding、DQN 双网络、经验回放）解决连续状态任务；
5. 说清"死亡三联症"（deadly triad）并能演示 off-policy 线性近似的发散；
6. 实现 REINFORCE（含 baseline）并用策略梯度算法（PPO 级）训练 Gymnasium 任务；
7. 把 TD 误差与多巴胺、Rescorla–Wagner 模型、TD-Gammon、AlphaGo 串成一条"从理论到应用"的线；
8. 交付一个完整 RL 项目：含实验设计、学习曲线、消融/对比、文档与答辩演示。

### 先修要求

- Python 3 + NumPy（矩阵运算、随机数）+ Matplotlib（画曲线）
- 概率基础：期望、条件概率、简单贝叶斯（能读懂 $P(b|a)$ 级别的东西即可）
- 线性代数基础：向量、矩阵乘法、梯度（能看懂 $\nabla_\theta$ 记号）
- 无需 RL 经验；会一点 C++ 更好（读 SB3 源码用）

## 2. 教材与辅助材料怎么配合

| 用途 | 材料 |
|---|---|
| 理论主线（必读） | R 书：每章节 + 节标题（下方各周已标 PDF 页码） |
| 快速直觉 + 现代代码 | OpenAI Spinning Up: https://spinningup.openai.com/ |
| 可直接跑的算法库 | stable-baselines3（DQN/PPO 基线对比）: https://stable-baselines3.readthedocs.io/ |
| 任务环境 | Gymnasium: https://gymnasium.farama.org/ |

**阅读规则**：先按"节标题"扫一遍建立地图 → 精读公式与证明思路（不必逐字）→ 做实验 → 用"自测"问题验收。R 书的 `*` 号节（如 *Off-policy Methods with Approximation）可放在第二遍读。

| 课程主题 | R 书章节（PDF 页码） |
|---|---|
| 什么是 RL、agent–environment | Ch1 Introduction (p.23–44) |
| 多臂老虎机 / 探索-利用 | Ch2 Multi-armed Bandits (p.47–68) |
| MDP、回报、价值函数、Bellman 方程 | Ch3 Finite MDPs (p.69–94) |
| 动态规划（DP） | Ch4 Dynamic Programming (p.95–112) |
| 蒙特卡洛（MC） | Ch5 Monte Carlo Methods (p.113–140) |
| TD 学习、Sarsa、Q-learning | Ch6 Temporal-Difference Learning (p.141–162) |
| n-step bootstrapping | Ch7 n-step Bootstrapping (p.163–180) |
| 模型、Dyna、规划、MCTS | Ch8 Planning and Learning with Tabular Methods (p.181–216) |
| 函数近似（线性、tile coding、ANN） | Ch9 On-policy Prediction with Approximation (p.219–264) |
| 半梯度控制 / DQN / 平均回报 | Ch10 On-policy Control with Approximation (p.265–278) |
| off-policy 近似、死亡三联症 | Ch11 *Off-policy Methods with Approximation (p.279–308) |
| 资格迹、TD(λ)、Sarsa(λ) | Ch12 Eligibility Traces (p.309–342) |
| 策略梯度、REINFORCE、Actor-Critic | Ch13 Policy Gradient Methods (p.343–360) |
| 心理学（Rescorla–Wagner、TD 模型） | Ch14 Psychology (p.363–398) |
| 神经科学（多巴胺、RPE） | Ch15 Neuroscience (p.399–442) |
| 应用案例（TD-Gammon、AlphaGo、Atari） | Ch16 Applications and Case Studies (p.443–480) |
| 前沿（GVF、options、reward 设计） | Ch17 Frontiers (p.481–502) |

## 3. 16 周总表

| 周 | 模块 | 主题 | 阅读 | 里程碑 |
|---|---|---|---|---|
| 1 | M1 基础 | 什么是 RL + 环境搭建 | Ch1 | 环境 + 第一个随机 agent |
| 2 | M1 基础 | 多臂老虎机 | Ch2 | 10-arm 实验（4 种选择策略） |
| 3 | M2 表格法 | MDP 与价值函数 | Ch3 | 手工算 3 状态 MDP |
| 4 | M2 表格法 | 动态规划 | Ch4 | gridworld DP |
| 5 | M2 表格法 | 蒙特卡洛 | Ch5 | MC vs DP 对比 |
| 6 | M2 表格法 | TD / Sarsa / Q | Ch6 | **项目一**（4 算法对比） |
| 7 | M2 表格法 | n-step bootstrapping | Ch7 | n 扫描曲线 |
| 8 | M2 表格法 | Dyna / 规划 / MCTS | Ch8 | 阶段考 2 + MCTS 小游戏 |
| 9 | M3 深度 RL | 函数近似（线性/ANN） | Ch9 | MountainCar 近似法 |
| 10 | M3 深度 RL | DQN / 半梯度控制 | Ch10 | Atari 或 CartPole DQN |
| 11 | M3 深度 RL | 死亡三联症 / off-policy 近似 | Ch11 | 发散复现实验 |
| 12 | M3 深度 RL | 资格迹 TD(λ) | Ch12 | λ 扫描 + 稳定性 |
| 13 | M3 深度 RL | 策略梯度 / Actor-Critic | Ch13 | **项目二**（PPO/DQN 对比） |
| 14 | M4 深入 | 心理学 + 神经科学 | Ch14–15 | Rescorla–Wagner/TD 模拟 |
| 15 | M4 深入 | 应用案例 + 毕业开题 | Ch16–17 | 开题评审 |
| 16 | M4 深入 | 毕业项目答辩 | — | **最终项目** |

## 4. 分模块详细计划

> 每周的**课件**见侧边栏 "RL Courseware" 分区（`RL-Courseware/` 目录）：对照书本阅读 + 按课件"动手步骤"做实验，再用每页末尾的"笔记清单"补齐本章笔记。

### 模块一 · 基础（W1–W2）

#### W1 · 什么是 RL
- **目标**：说清 RL 与监督/无监督学习的区别；掌握 agent–environment 接口与 reward hypothesis；搭好实验环境。
- **阅读**：Ch1 (p.23–44)，重点 1.2 Examples、1.3 Elements、1.5 Tic-Tac-Toe。
- **实验**：Python 环境 + Gymnasium 跑通 `CartPole-v1` 随机 agent；`FrozenLake-v1` 看 episode 结构。
- **交付物**：环境清单 + 随机 agent 的 10 局结果截图。
- **自检**：RL 三要素？什么是 episode？reward 与 return 的区别？

#### W2 · 多臂老虎机
- **阅读**：Ch2 (p.47–68)，重点 2.3–2.7（10-armed testbed、nonstationary、optimistic、UCB、gradient bandit）。
- **实验**：实现 10-armed testbed；对比 ε-greedy / optimistic / UCB / gradient bandit 的 cumulative reward 曲线；再做一次 nonstationary 场景。
- **交付物**：4 条曲线图 + 参数消融（ε=0.1 vs 0.01）+ 结论 3 句话。
- **自检**：为什么 incremental update 不用存所有样本？UCB 的 "confidence" 从何而来？

### 模块二 · 表格方法（W3–W8）

#### W3 · 有限 MDP
- **阅读**：Ch3 (p.69–94)，重点 3.2 Goals and Rewards、3.4 Policies/Value、3.5 Optimal、3.8 Bellman equations（章末汇总）。
- **实验**：画一个 3–5 状态的 MDP；手算 + 代码算 V_π、q_π；验证 Bellman 方程两边相等。
- **自检**：episodic 与 continuing 任务的 G_t 公式差异？V_π 和 q_π 的关系？

#### W4 · 动态规划
- **阅读**：Ch4 (p.95–112)，重点 4.2 策略评估、4.3 策略改进、4.4/4.5 policy/value iteration、4.6 GPI。
- **实验**：4×4 gridworld（风/随机转移）跑 policy iteration 与 value iteration；画策略演化。
- **自检**：为什么 DP 需要"完整模型"？GPI 循环的两条腿是什么？

#### W5 · 蒙特卡洛
- **阅读**：Ch5 (p.113–140)，重点 5.1–5.5（MC prediction/control、without ES、importance sampling）。
- **实验**：同一 gridworld 上 MC control（ES 与 ε-soft）；与 W4 的 DP 结果叠图对比。
- **自检**：MC 为什么必须等 episode 结束？off-policy 的 IS 权重为什么容易爆炸？

#### W6 · TD 学习 + 项目一
- **阅读**：Ch6 (p.141–162)，重点 TD(0)、Sarsa、Q-learning、Expected Sarsa、double Q。
- **项目一 · 四算法对比实验**（同一随机 gridworld）：
  - 实现 DP / MC / Sarsa / Q-learning 四个 control 算法
  - 输出：初始→最优的 value map 演化（4 帧）、cumulative reward 曲线、"到达最优策略"的 episode 数
  - 写 1 页结论：哪个算法最快、为什么、各自的适用条件
- **自检**：TD 更新为什么是"semi-gradient"？Sarsa 与 Q-learning 在"悬崖问题"上的行为差异？

#### W7 · n-step bootstrapping
- **阅读**：Ch7 (p.163–180)，重点 n-step TD、n-step Sarsa、Tree Backup、λ 视角（n=1 到 n=∞ 的谱）。
- **实验**：CartPole 或 gridworld 上扫 n=1,2,4,8,∞，画收敛速度对比。
- **自检**：n-step 在 sample efficiency 与 bias 上如何权衡？Tree Backup 解决了 off-policy n-step 的什么问题？

#### W8 · 规划：Dyna 与 MCTS + 阶段考 2
- **阅读**：Ch8 (p.181–216)，重点 Dyna、Prioritized Sweeping、RTDP、Rollout、MCTS（8.9–8.11）。
- **实验**：① Dyna-Q vs Q-learning 同任务对比；② 实现一个玩具 MCTS 下井字棋（tic-tac-toe）并击败随机玩家。
- **阶段考 2**：Bellman 方程推导题、Sarsa/Q 更新式默写、IS 权重、Dyna 流程、"表格法 vs 近似法"边界。
- **自检**：模型错了 Dyna 会怎样（Ch8.4）？MCTS 的 select/expand/simulate/backprop 四步？

### 模块三 · 深度强化学习基础（W9–W13）

#### W9 · 函数近似：预测
- **阅读**：Ch9 (p.219–264)，重点 9.3 stochastic/semi-gradient、9.4 线性方法、9.5 tile coding、9.7 神经网络、9.9 LSTD。
- **实验**：`MountainCarContinuous-v0` 或离散 MountainCar：线性特征（tile coding）TD 预测 + 小网络版本；对比曲线。
- **自检**：semi-gradient 与普通 gradient 的区别（目标也在变）？tile coding 为什么能外推？

#### W10 · 半梯度控制：DQN
- **阅读**：Ch10 (p.265–278)，重点 10.2 semi-gradient n-step Sarsa、10.4（2nd ed. 的 DQN 要点：replay、target network）。
- **实验**：Gymnasium `CartPole-v1` / `BreakoutNoFrameskip-v4`：手写简化 DQN（replay buffer + target net）并与 SB3 的 DQN 对比。
- **自检**：target network 为什么能稳住训练？replay 打散了什么？

#### W11 · off-policy 近似与"死亡三联症"
- **阅读**：Ch11 (p.279–308)，重点 11.2 发散例子、11.3 deadly triad、11.5–11.7（Bellman error、gradient-TD）。
- **实验**：复现"线性 + bootstrapping + off-policy"发散（书 11.2 的场景）；记录发散曲线。
- **自检**：三件套各是什么？DQN 的哪两个工程手段正好破掉其中两件？

#### W12 · 资格迹
- **阅读**：Ch12 (p.309–342)，重点 λ-return、TD(λ)、True Online TD(λ)、Sarsa(λ)、Q(λ) vs TB(λ) 谱系。
- **实验**：λ 扫描（0, 0.25, 0.5, 0.75, 0.9, 1）在 CartPole/MountainCar 上的收敛对比。
- **自检**：λ=0 与 λ=1 分别退化成什么？eligibility trace 的直觉（"最近访问过的状态多分点功劳"）？

#### W13 · 策略梯度 + 项目二
- **阅读**：Ch13 (p.343–360)，重点 13.2 PG theorem、13.3 REINFORCE、13.4 baseline、13.5 actor-critic。
- **项目二 · 你的第一个"深度 RL"实验**（二选一任务：`LunarLander-v3` 或 `HalfCheetah` 简化版）：
  - 用 SB3 训练 DQN（若动作离散）与 PPO（连续动作），固定种子 ≥3 个
  - 输出：test 曲线（滑动平均）、最优/最差轨迹 gif、超参消融（LR、entropy coef 或 γ）
  - 报告：1 页结论 + "什么情况下该选 value-based vs policy-based"
- **阶段考 3**：PG theorem 推导（给提示链）、REINFORCE 方差来源、baseline 为什么 unbiased、DQN 双网络机制。
- **自检**：REINFORCE 为什么高方差？baseline 减去 V(s) 后为什么仍无偏？

### 模块四 · 从脑到应用（W14–W16）

#### W14 · 心理学与神经科学
- **阅读**：Ch14 (p.363–398) 重点 14.3 Rescorla–Wagner 与 TD 模型；Ch15 (p.399–442) 重点 15.4 dopamine、15.6 TD error 对应。
- **实验**：实现 Rescorla–Wagner 的 "blocking 效应" 模拟（条件刺激 A/B，先学 A 再学 AB），画预测值曲线；再用 TD 模型复现"延迟强化"的 RPE 波形。
- **自检**：blocking 效应说明了什么？多巴胺 RPE 信号与 TD error 的对应证据？

#### W15 · 应用案例 + 毕业开题
- **阅读**：Ch16 (p.443–480) 重点 TD-Gammon、AlphaGo(Zero)、reward 设计；Ch17 Frontiers 中 "Designing Reward Signals" (p.491–494)。
- **交付物**：开题报告（1 页：目标、架构图、4 个里程碑 + 验收标准、风险预案、rubric 自评）。
- **评审**：按 W16 课件的"评审检查清单"逐条过。

#### W16 · 毕业项目答辩
- 10 分钟演示 + 10 分钟问答；提交 repo（数据/曲线可复现）+ 报告 + 视频。
- 阅读收尾：Ch17 其余节（GVF、options、RL 与 AI 未来）。

## 5. 考核方式

| 项目 | 占比 | 说明 |
|---|---|---|
| 每周实验 | 25% | 按各周"交付物"打分，取平均 |
| 阶段考 | 15% | 三次（W8、W13、W16 前），各 5% |
| 项目一 | 15% | W6 四算法对比 |
| 项目二 | 20% | W13 深度 RL 实验报告 |
| 毕业项目 | 15% | W16，含答辩 |
| 平时表现 | 10% | 代码互评、提问、学习日志 |
| **合计** | **100%** | ≥60 分及格 |

- **迟交**：每天 −10%，超过 3 天不接受。
- **学术诚信**：手写核心算法（DP/MC/TD/Sarsa/Q 必须自己写）；用 SB3 做基线对比必须声明。

## 6. 毕业项目选题（任选其一）

| 编号 | 项目 | 核心栈 | 必做 | 加分项 |
|---|---|---|---|---|
| A | MCTS 游戏 AI | Ch8 + 自己的树搜索 | 四子棋/五子棋 MCTS 击败规则引擎 | 置换表、PUCT 调参、自对弈 |
| B | 机器人迷宫 RL | Gym 环境 + DQN/PPO | 连续控制任务训练 + 策略评估 | 与规划法（A*/Nav2 思路）对比 |
| C | Reward 设计研究 | Ch16–17 | 同一任务 3 种 reward 设计对比 + reward hacking 案例 | 潜在基奖励/逆 RL 概念实现 |
| D | 脑模拟：TD 与多巴胺 | Ch14–15 | Rescorla–Wagner blocking + TD 延迟强化仿真 | 与电生理数据（文献）对照讨论 |
| E | Contextual Bandit | Ch2 + 线性模型 | 模拟 A/B 场景：LinUCB vs 随机 vs 最优 | 真实网页点击数据集 |

**评分量规（rubric）**：功能完整性 35% / 实验设计与曲线质量 25% / 代码质量 20% / 文档与答辩 20%。

## 7. 自学模式（Self-study Adaptation）

1. **验收门**：上一里程碑的曲线/结论没出来，不进入下一周（用"自检"问题自测）。
2. **标准节奏（16 周）**：每周 8–12 小时 —— 2.5h 阅读、3h 手写算法、2–4h 跑实验 + 画图、1h 写笔记。
3. **紧凑节奏（8 周）**：每周 16–24 小时，按两周合并（W1-2 / W3-4 / W5-6 / W7-8 / W9-10 / W11-12 / W13-14 / W15-16）。
4. **学习日志**：每章笔记建议按 `Chapter-N-<主题>.md` 命名（见下方"建议笔记清单"），沿用本站"英文原文 + `.zh.md`"习惯。
5. **模拟答辩**：W16 录 10 分钟演示视频，对照 rubric 自评；<80 分返工一次。

## 8. 环境清单

- [ ] Python 3.10+（venv）
- [ ] `pip install numpy matplotlib gymnasium stable-baselines3 torch`（torch 仅项目二需要）
- [ ] Jupyter（可选，画曲线/调试更顺手）
- [ ] 4 核 CPU 即可跑完 W1–W13；DQN(Atari) 建议 8 核或 2 小时以上时间预算
- [ ] （可选）GPU：加速 Atari / PPO 训练
- [ ] git 仓库：`rl-course/`（按周建子目录 `w01/ ... w16/`）

## 9. 建议笔记清单（写作即学习）

| 周 | 建议笔记文件名 | 对应书章节 |
|---|---|---|
| W1 | `Chapter-1-Introduction.md` | Ch1 |
| W2 | `Chapter-2-Multi-Armed-Bandits.md` | Ch2 |
| W3 | `Chapter-3-Finite-MDPs.md` | Ch3 |
| W4 | `Chapter-4-Dynamic-Programming.md` | Ch4 |
| W5 | `Chapter-5-Monte-Carlo-Methods.md` | Ch5 |
| W6 | `Chapter-6-Temporal-Difference-Learning.md` | Ch6 |
| W7 | `Chapter-7-n-step-Bootstrapping.md` | Ch7 |
| W8 | `Chapter-8-Planning-and-Dyna.md` | Ch8 |
| W9 | `Chapter-9-Function-Approximation.md` | Ch9 |
| W10 | `Chapter-10-DQN-and-Semi-gradient.md` | Ch10 |
| W11 | `Chapter-11-Deadly-Triad.md` | Ch11 |
| W12 | `Chapter-12-Eligibility-Traces.md` | Ch12 |
| W13 | `Chapter-13-Policy-Gradient.md` | Ch13 |
| W14 | `Chapter-14-15-Brain-and-RL.md` | Ch14–15 |
| W15 | `Chapter-16-Applications.md` | Ch16 |
| W16 | `Chapter-17-Frontiers-and-Recap.md` | Ch17 |

## 10. 补充资源

- OpenAI Spinning Up（直觉 + PyTorch 实现）: https://spinningup.openai.com/
- stable-baselines3 文档与示例: https://stable-baselines3.readthedocs.io/
- Sutton & Barto 官网（2nd edition 说明）: https://incompleteideas.net/book/the-book-2nd.html
- 经典论文（选读）：Mnih et al. 2015 (DQN, Nature)、Silver et al. 2016/2017 (AlphaGo/Zero)、Williams 1992 (REINFORCE)
- 中文辅助：B站/YouTube 的 "Reinforcement Learning" 系列课程（找 Sutton 原书逐章讲解版）
