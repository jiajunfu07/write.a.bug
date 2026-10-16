# W02 · 多臂老虎机（课件）

> 对应阅读: Ch2 Multi-armed Bandits (p.25–46)，重点 2.3–2.7
> 里程碑: 10-armed testbed 四条曲线 + 结论
> 配套笔记: 建议 `Chapter-2-Multi-Armed-Bandits.md`（待写）

## 学习目标

1. 掌握 action-value 方法 + ε-greedy 的完整实现；
2. 理解 incremental update 为什么是 O(1) 存储；
3. 会做 10-armed testbed 实验，读懂"cumulative reward 曲线"；
4. 解释 optimistic initial values、UCB、gradient bandit 的动机与机制。

## 核心概念

### 1. k-armed bandit 问题（2.1）

每个动作 a 有未知均值 `q*(a)`（本课件用确定性 + 随机两种版本）。目标：最大化平均 reward = 尽量少选差臂、同时保持探索。

### 2. Action-value + ε-greedy（2.2）

- 维护每个臂的估计 `Q_t(a)`；
- **ε-greedy**：以概率 1−ε 选当前最大 Q（利用），以概率 ε 随机选（探索）；
- **incremental update**（2.4，必背）：

```
Q_{t+1}(a) = Q_t(a) + [1/t] * (r_t - Q_t(a))
```

  等价于"均值 = 上一均值 + 1/样本数 × (新样本 − 上一均值)"，不需要存所有样本。

### 3. 10-armed testbed（2.3）

10 个臂的真实值 `q*(a)` 从 N(0,1) 采样；每次选择 reward ~ N(q*(a), 1)。**所有算法用同一个 q* 向量** → 曲线可比。

### 4. 三种"聪明"探索（2.6–2.8）

| 方法 | 机制 | 直觉 |
|---|---|---|
| optimistic initial | 所有 Q 初始化为 5（偏大） | 每个臂都像"潜力股"，必被试到；越差越早被放弃 |
| UCB | `a = argmax Q_t(a) + c*sqrt(ln t / N_t(a))` | "乐观 + 不确定性奖励"；没试过的臂 bonus 大 |
| gradient bandit | 维护 h_t = Q_t − Q_avg，更新 `h += α(r − h_avg − Σh)∇log π` | 学"相对好坏"，天然平衡探索（2.8） |

### 5. Nonstationary（2.5）

环境均值随时间漂移时：固定 α 的 incremental update（滑动窗口）比 1/t（全历史平均）更能跟上。→ 记住这句话，W14 神经科学会用上。

## 动手步骤（实验代码骨架）

```python
import numpy as np
import matplotlib.pyplot as plt

def make_env(seed=0):
    rng = np.random.default_rng(seed)
    q_true = rng.normal(0, 1, 10)
    return q_true, rng

def run(q_true, rng, T=1000, k=10, mode="eps_greedy", eps=0.1,
        c_ucb=1.0, optimistic=5.0):
    Q = np.full(k, optimistic) if optimistic > 0 else np.zeros(k)
    N = np.zeros(k)
    hist = np.zeros(T)
    for t in range(T):
        if mode == "eps_greedy" and rng.random() < eps:
            a = rng.integers(k)
        elif mode == "ucb":
            N_safe = np.maximum(N, 1)
            bonus = c_ucb * np.sqrt(np.log(t + 1) / N_safe)
            a = int(np.argmax(Q + bonus))
        else:
            a = int(np.argmax(Q))
        r = rng.normal(q_true[a], 1.0)
        N[a] += 1
        Q[a] += (1 / N[a]) * (r - Q[a])          # incremental update
        hist[t] = r
    return np.cumsum(hist)

# 跑 4 种策略 × 多条种子, 画 cumulative reward 曲线
```

验收：4 条平均曲线（eps_greedy / optimistic / ucb / random），标注最优臂的累计 reward 作上界参考。

## 常见坑

- **不同算法用了不同的 q\* 采样** → 曲线不可比（必须同一 seed 的 q_true + 同一 reward 随机流结构）。
- UCB 里 `N=0` 除零 → 先初始化 N≥1 或用 optimistic Q 让每个臂先被选一次。
- optimistic initial 设得太小（如 0.1）→ 探索不足，曲线看起来和 ε-greedy 差不多。
- 把 `1/t` 更新用在 nonstationary 场景 → 曲线在漂移后"追不上"，这正是 2.5 的要点。
- 画曲线用"单条 episode"而不是"多 episode 平均" → 噪声大，结论不稳。

## 自测

1. 为什么 incremental update 是 O(1) 存储？写出递推式。
2. ε-greedy 的 ε 太大/太小各有什么问题？
3. optimistic initial values 为什么"保证"每个臂至少被试一次？
4. UCB 的 bonus 项 `sqrt(ln t / N(a))` 随什么变大/变小？
5. nonstationary 场景为什么用固定 α？

### 参考答案

1. 只存 (Q, N)：Q ← Q + (1/N)(r − Q)；不需要历史样本列表。
2. 太大→浪费在随机臂上（under-utilization）；太小→过早收敛到次优臂（over-commitment，尤其乐观初始化没及时衰减时）。
3. 初始 Q 高于真实期望 → 选它期望"失望"，但没被选过的臂 Q 始终最高 → 必被试到（在 Q 均值回归到真实值之前）。
4. t 越大、N(a) 越小，bonus 越大；试得越多 bonus 越小（不确定度下降）。
5. 1/t 把全历史平均权重给早期样本，环境变了估计被"锁死"；固定 α 相当于指数滑动窗，能跟上漂移。

## 笔记清单（对照书本补写）

- [ ] 抄写 incremental update 推导（从均值定义推出递推式）
- [ ] 记录 10-armed testbed 实验参数表（ε、c、optimistic 值 × 曲线结论）
- [ ] 用自己的话写"探索-利用权衡"的 3 个例子（bandit / 餐厅 / 广告）
- [ ] 整理 UCB 公式 + 每个符号含义
- [ ] 写一段"gradient bandit 为什么天然平衡探索"（2.8 的直觉）
