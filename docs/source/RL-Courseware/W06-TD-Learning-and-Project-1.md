# W06 · TD 学习（Sarsa/Q-learning）+ 项目一（课件）

> 对应阅读: Ch6 Temporal-Difference Learning (p.119–140)，重点 TD(0)、Sarsa、Q-learning、Expected Sarsa、double Q
> 里程碑: **项目一 · 四算法对比** + 阶段考 1
> 配套笔记: 建议 `Chapter-6-Temporal-Difference-Learning.md`（待写）

## 学习目标

1. 理解 TD 的"bootstrap"思想与 semi-gradient 更新；
2. 默写 Sarsa 与 Q-learning 的更新式，说清 on/off-policy 之别；
3. 解释 maximization bias 与 double Q-learning 的修复；
4. 按验收清单交付项目一。

## 核心概念

### 1. TD(0)（6.2，全书最核心的一个式子）

MC 要等 episode 结束；TD 用"当前估计"代替"完整 return"：

```
TD target  = R_{t+1} + γ V(S_{t+1})
V(S_t) ← V(S_t) + α [ R_{t+1} + γ V(S_{t+1}) - V(S_t) ]
```

- **bootstrap**：用自身的（部分）估计去更新自己；
- **semi-gradient**：更新目标里的 V(S_{t+1}) 也在变（不是固定的最小二乘目标），所以"半个梯度"；
- 优点：每步可更新、在线、O(1) 存储 → 真实机器人只能这样学。

### 2. Sarsa（6.5，on-policy）

状态-动作-奖励-状态-**动作** 五元组，更新用"实际选择的下一个动作 a'"：

```
Q(s,a) ← Q(s,a) + α [ r + γ Q(s', a') - Q(s,a) ]
```

### 3. Q-learning（6.6，off-policy）

更新用"贪心的最大 Q"，与行为无关：

```
Q(s,a) ← Q(s,a) + α [ r + γ max_{a'} Q(s', a') - Q(s,a) ]
```

- 可以用 ε-greedy 探索，但学的是贪心策略 → 经典 off-policy；
- **Sarsa vs Q 的行为差异（悬崖问题 6.7）**：Sarsa 会"绕着悬崖走"（怕掉下去的期望惩罚）；Q-learning 走直线（掉下去的代价在 Q 里只计一次，贪心仍选直线）。这是理解 on/off-policy 的最佳例子。

### 4. Maximization bias 与 Double Q（6.9）

- max 是"有偏估计"（总偏向乐观）→ Q 系统性高估；
- **Double Q-learning**：两套 Q1/Q2，一个选 argmax、另一个评估：
  `target = r + γ Q2(s', argmax_a Q1(s',a))` → 偏差大幅降低。

### 5. Expected Sarsa（6.8）

用"策略期望"代替实际 a'：`r + γ Σ_a π(a|s')Q(s',a)` → 无需采样下一动作（适合 afterstate 结构）。

## 动手步骤

1. 在 W4/W5 的 gridworld 上实现 TD(0) 预测、Sarsa、Q-learning（代码结构与 W5 MC 类似，只是**每步更新**而不是 episode 末更新）；
2. **悬崖问题**（6.7）：1×5 走廊 + 两侧悬崖，起点左一、目标右一：
   - Sarsa 与 Q-learning 各跑 500 episode，画 episode 平均 return 曲线；
   - 记录两者最优路径差异（直线 vs 绕行）。
3. Double Q：加一套 Q2，对比 Q 的"高估程度"（统计 Q 均值 vs 真实最优 q*）。

## 项目一验收清单

- [ ] 同一随机 gridworld 上实现 **DP / MC / Sarsa / Q-learning** 四个 control 算法
- [ ] 输出 1：value map 演化 4 帧图（初始 → 33% → 66% → 收敛）× 4 算法
- [ ] 输出 2：cumulative reward 曲线（log y 轴）
- [ ] 输出 3：表格 —— 收敛 episode 数 / 最终策略正确率
- [ ] 1 页结论：谁最快、为什么、各自适用条件（模型已知/未知、在线/离线）
- [ ] README：复现命令 + 参数表

## 阶段考 1 范围（W2–W6）

- ε-greedy / UCB / optimistic 的动机与公式（2 题）
- Bellman 方程推导 + V*/q* 关系（2 题）
- MC 的 ES/ε-soft/IS 权重（2 题）
- Sarsa vs Q 更新式默写 + 悬崖问题行为解释（2 题）
- 代码挑错：给一段 TD 更新代码找 3 个 bug

## 常见坑

- **Sarsa 写成了 Q-learning**（更新里用了 max 而不是 a'）→ 两者曲线完全一样，说明你实现错了（应看悬崖问题差异）。
- 更新时机错：TD 应在"收到 r,s' 之后立即更新"，而不是 episode 末。
- α 恒定 vs 衰减：恒定 α 在非平稳任务更稳，平稳任务收敛后抖；对比一下。
- Q 的 key 是 (s,a) 二元组，别写成 s（字典会互相覆盖）。
- 终止状态（terminal）没有 max：更新 target 直接是 r（γ 项为 0），别查 Q(terminal)。

## 自测

1. 什么是 bootstrap？MC、TD、DP 各自 bootstrap 吗？
2. 默写 Sarsa 与 Q-learning 更新式，指出唯一差异项。
3. 为什么 Q-learning 是 off-policy 而 Sarsa 是 on-policy？
4. 悬崖问题中 Sarsa 为什么绕路？
5. double Q-learning 如何降低 maximization bias？

### 参考答案

1. 用当前估计代替真实目标值来更新。TD 是（用 V(s')）；MC 否（用完整 return）；DP 是（用 V(s')，但精确求期望）。
2. Sarsa: r + γQ(s',a')；Q: r + γmax Q(s',·)；差异就在"下一个动作"用实际的还是贪心的。
3. Q 学的是"贪心策略"的价值，与实际执行策略（ε-greedy）分离；Sarsa 学的就是当前行为策略。
4. Sarsa 的 target 用实际 a'：贴悬崖走时，探索性掉下去的惩罚直接计入 Q，绕路期望更高；Q 的 max 只挑最好后继，直线策略的"期望"仍最高。
5. argmax 由 Q1 选、价值由 Q2 评估，两个独立高估不会同时落在同一动作上 → 组合估计更接近真实。

## 笔记清单（对照书本补写）

- [ ] 抄写 TD(0) / Sarsa / Q-learning / double Q 四个更新式 + 逐符号注释
- [ ] 画悬崖问题的两条路径图 + 行为解释（各 2 句）
- [ ] 用自己的话写"bootstrap 为什么是 RL 的核心 trick"
- [ ] 记录项目一四算法对比表 + 你的结论
- [ ] 整理 on-policy / off-policy 术语表（行为策略 vs 目标策略）
