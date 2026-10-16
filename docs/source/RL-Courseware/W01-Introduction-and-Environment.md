# W01 · 什么是 RL + 环境搭建（课件）

> 对应阅读: Ch1 Introduction (p.1–22)，重点 1.2 Examples、1.3 Elements、1.5 Tic-Tac-Toe
> 里程碑: 环境跑通 + 随机 agent · 交付: 环境清单 + 10 局结果截图
> 配套笔记: 建议 `Chapter-1-Introduction.md`（待写）

## 学习目标

1. 用自己的话说清 RL 与监督/无监督学习的区别；
2. 掌握 agent–environment 接口、episode、reward、return 四个术语；
3. 说出 reward hypothesis 的两个"敢想"之处；
4. 搭好 Python/Gymnasium 环境并跑通第一个（随机）agent。

## 核心概念

### 1. 三要素（1.3 Elements）

```
        state s_t  +  reward r_{t+1}
environment ─────────────────────────▶ agent
        ▲                              │
        └──────────── action a_t ──────┘
```

- **Agent** 在每一步 t 观察到 **state**（或 partial observation），执行 **action**；
- 环境返回 **reward**（标量，0/1 或 -1/+1 居多）+ 下一个 state；
- **Episode** = 一条从初始状态到终止状态（或最大步数）的轨迹：
  `S0, A0, R1, S1, A1, R2, ...`
- **Return**（回报）G_t = 从 t 起未来 reward 的折现和：
  `G_t = R_{t+1} + γ R_{t+2} + γ² R_{t+3} + ...`（γ ∈ [0,1)）

### 2. 与监督/无监督的区别（1.1）

| | 监督 | 无监督 | RL |
|---|---|---|---|
| 反馈 | 正确答案（label） | 无 | 分数（reward），且**有延迟** |
| 探索 | 无 | 无 | 需要（探索-利用） |
| 目标 | 拟合映射 | 发现结构 | 学一个**行为策略**使长期 return 最大 |

关键差异：RL 的"教师"只打分、不教动作；而且反馈是延迟的（第 100 步撞墙，锅在哪个动作？——这是后面整本书要解决的问题）。

### 3. Reward Hypothesis（1.2，全书公理）

> 所有成功都可以表述为**最大化累计预期 reward**。

两个大胆之处：
1. "成功"（success）能且只需用一个标量累计分数刻画；
2. "最大化"（maximizing）是唯一的动机形式（不需要规则、不需要"目标达成"语义）。

### 4. Tic-Tac-Toe 例子（1.5）为什么重要

- 展示 episode 结构（最多 9 步）、win/loss/draw 的 reward 设计；
- 状态空间只有 3^9=19683 种 → 可以**枚举所有局面**（表格法可行的规模感）；
- 后面 W8 会用 MCTS 把它"打败"。

## 动手步骤

1. **环境**：

```bash
python3 -m venv rlvenv && source rlvenv/bin/activate
pip install numpy matplotlib gymnasium
# 项目二(深度RL)再装: pip install torch stable-baselines3
```

2. **第一个 agent（随机策略）**：

```python
import numpy as np
import gymnasium as gym

env = gym.make("CartPole-v1", render_mode=None)
total = 0
for ep in range(10):
    obs, _ = env.reset(seed=ep)
    done = False
    while not done:
        a = env.action_space.sample()      # 随机动作
        obs, r, terminated, truncated, _ = env.step(a)
        total += r
        done = terminated or truncated
    print(f"episode {ep}: reward={total}")
env.close()
```

3. 再看 `FrozenLake-v1`（episodic、稀疏 reward）：观察 reward 只在到达 G 时才非零。
4. 记录：每个环境的 state/action 空间大小（`env.observation_space` / `env.action_space`）。

## 常见坑

- `gym`（旧版）与 `gymnasium`（新版）API 不同：本书示例是 2020 年的，环境部分以 Gymnasium 为准（step 返回 5 个值）。
- 忘了 `env.reset(seed=...)` → 每次运行初始状态不同，曲线不可复现。
- CartPole 的 reward 是"活了多少步"（每步 +1），不是 0/1——稀疏 vs 密集 reward 的第一课。
- 在循环外忘记 `env.close()` → 资源泄漏（小问题，但养成习惯）。

## 自测

1. RL 三要素 + 每一步的信息流（画出来）。
2. episode、trajectory、return 三个词的区别？
3. reward hypothesis 的两个大胆之处？
4. 为什么 RL 必须面对"探索-利用"问题，而监督学习不需要？
5. Tic-Tac-Toe 的状态空间有多大？为什么这让它适合做表格法练习？

### 参考答案

1. agent、environment、（隐式的）time；agent 发 action，环境回 state+reward，循环往复。
2. episode=一条完整轨迹（终止边界）；trajectory=状态-动作-奖励序列（可指 episode 内的一段）；return=从某时刻起未来 reward 的折现和。
3. 一切成功都能写成"最大化累计标量 reward"；"最大化"是唯一动机形式。
4. 监督学习有"标准答案"直接学映射；RL 只有延迟分数，不试（探索）就不知道哪个动作好（利用当前认知）。
5. 3^9=19683（每格空/X/O 三种 × 9 格），加上合法性约束更少——表格法可枚举。

## 笔记清单（对照书本补写）

- [ ] 画 agent–environment 接口图 + 术语表（state/action/reward/episode/return/policy）
- [ ] 用自己的话写 reward hypothesis（≥3 句）+ 一个你觉得它"站不住"的例子
- [ ] 记录 Gymnasium 环境的 step/reset API 与旧版 gym 的差异
- [ ] 整理"RL vs 监督 vs 无监督"对比表
- [ ] 截图：随机 agent 10 局结果 + FrozenLake 一局轨迹
