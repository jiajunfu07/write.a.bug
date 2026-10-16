# W03 · 有限 MDP 与价值函数（课件）

> 对应阅读: Ch3 Finite Markov Decision Processes (p.47–72)，重点 3.2/3.4/3.5 + 章末 Bellman equations 汇总
> 里程碑: 3–5 状态 MDP 的 V/q 手算 + 代码验证
> 配套笔记: 建议 `Chapter-3-Finite-MDPs.md`（待写）

## 学习目标

1. 定义 MDP 五元组，说明"马尔可夫性"意味着什么；
2. 区分 V_π、q_π、V*、q*，会写它们的 Bellman 方程；
3. 理解 episodic vs continuing 的 return 差异；
4. 能用代码验证 Bellman 方程两边相等。

## 核心概念

### 1. MDP 五元组（3.1）

`(S, A, R, P, γ)`：状态集、动作集、奖励函数 `R(s,a,s')`（或 R(s,a)）、转移概率 `P(s'|s,a)`、折扣 γ。

**马尔可夫性**：`P(s_{t+1}, r_{t+1} | s_t, a_t) 只依赖当前 (s_t, a_t)`——历史全部被当前状态"压缩"掉。这是能用表格/状态空间方法的前提。

### 2. 策略与价值函数（3.4）

- **策略** π(a|s)：状态到动作（分布）的映射。deterministic 策略 π(s)∈A。
- **state-value**（策略 π 下，从 s 出发的预期 return）：
  `V_π(s) = E_π[ G_t | S_t = s ]`
- **action-value**：
  `q_π(s,a) = E_π[ G_t | S_t=s, A_t=a ]`
- **最优**：存在一个策略使 V 在所有状态都最大，记 V*；q* 同理。
- 关系（必背）：
  `V_π(s) = Σ_a π(a|s) q_π(s,a)`
  `q_π(s,a) = R(s,a) + γ Σ_s' P(s'|s,a) V_π(s')`

### 3. Bellman 方程（3.8 / 章末汇总，全书心脏）

把 G_t 展开一步：`G_t = R_{t+1} + γ G_{t+1}`，取期望：

```
V_π(s) = Σ_a π(a|s) [ R(s,a) + γ Σ_s' P(s'|s,a) V_π(s') ]

V*(s)  = max_a [ R(s,a) + γ Σ_s' P(s'|s,a) V*(s') ]        (最优 Bellman 方程)
```

Bellman 方程的直觉：**"今天的价值 = 今天的奖励 + 明天的价值（折现）"**——递归结构 = 动态规划/所有 TD 方法的根基。

### 4. 为什么最优策略可以"只依赖当前状态"（3.5）

因为马尔可夫性：给定当前状态，过去不再影响未来；所以最优策略是**确定性的**且**记忆less 的**（一个查表）。

## 动手步骤

1. **画一个 3 状态 MDP**（手绘或 draw.io）：s1→(a)→s2 概率 0.7 / s3 概率 0.3，reward 标在箭头上，γ=0.9。
2. **手算**固定策略 π 的 V_π：写出 3 个 Bellman 方程（线性方程组）→ 解出 V。
3. **代码验证**（把 Bellman 方程变成矩阵）：

```python
import numpy as np

# V = R + γ P_π V   =>   (I - γ P_π) V = R
# R[s] = sum_a pi(a|s) sum_s' P(s'|s,a) R(s,a,s')
# P_π[s,s'] = sum_a pi(a|s) P(s'|s,a)
I = np.eye(3)
V = np.linalg.solve(I - 0.9 * P_pi, R_vec)
print(V)   # 与手算对比
```

4. **最优策略**：手写 value iteration 的 5 行版本（为 W4 预热）：

```python
V = np.zeros(3)
for _ in range(1000):
    V_new = R + 0.9 * (P @ V)     # 示意: 每个状态取 max_a
    if np.max(np.abs(V_new - V)) < 1e-6: break
    V = V_new
```

## 常见坑

- **R 的粒度**：`R(s,a)` 与 `R(s,a,s')` 两种定义混用 → Bellman 方程写错一项（先统一用 `R(s,a,s')`）。
- γ=1 的 continuing 任务：return 可能发散（reward 和无穷）——2nd ed. 后文改用 average reward（Ch10），先记住这个坑。
- 把 V(s) 当成"某次 episode 的实际得分" → V 是**期望**，不是单次值。
- 转移概率行不归一（Σ P ≠ 1）→ 矩阵解出来是错的。
- episodic 任务终止状态没给 V(terminal)=0 → 边界条件错。

## 自测

1. MDP 五元组各是什么？"马尔可夫性"一句话？
2. V_π 与 q_π 的定义 + 两者关系式？
3. 写出最优 Bellman 方程（V* 和 q* 各一个）。
4. 为什么最优策略是确定性的、无记忆的？
5. episodic 与 continuing 任务的 G_t 公式差在哪？

### 参考答案

1. (S, A, R, P, γ)；给定当前 (s,a)，未来与历史独立。
2. V_π(s)=E_π[G_t|S_t=s]；q_π(s,a)=E_π[G_t|S_t=s,A_t=a]；V_π(s)=Σ_a π(a|s)q_π(s,a)。
3. V*(s)=max_a [R(s,a)+γΣ P(s'|s,a)V*(s')]；q*(s,a)=R(s,a)+γΣ P(s'|s,a) max_{a'} q*(s',a')。
4. 马尔可夫性使"未来只依赖当前状态"，所以最优动作可以只查当前状态；取 max 必有一个确定性 argmax 最优（3.5 的证明思路）。
5. episodic 有终止（G_t 有限和）；continuing 是无穷和（需 γ<1 或 average reward 设定）。

## 笔记清单（对照书本补写）

- [ ] 画一个自己的 3 状态 MDP 图（带 reward 与概率）
- [ ] 抄写 4 个 Bellman 方程（V_π / q_π / V* / q*）+ 逐符号注释
- [ ] 记录矩阵法解 V_π 的代码 + 与手算的对比
- [ ] 用自己的话写"为什么 RL 的一切方法都在解 Bellman 方程"（3–5 句）
- [ ] 整理 episodic/continuing 对比表
