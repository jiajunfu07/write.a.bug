# W04 · 动态规划（课件）

> 对应阅读: Ch4 Dynamic Programming (PDF p.95–112)，重点 4.2–4.6 + 4x4 gridworld 例
> 里程碑: gridworld DP（policy iteration + value iteration）
> 配套笔记: 建议 `Chapter-4-Dynamic-Programming.md`（待写）

## 学习目标

1. 掌握"策略评估 → 策略改进"的 GPI 循环；
2. 实现 policy iteration 与 value iteration，并说清两者差异；
3. 在随机 gridworld 上画出策略演化过程；
4. 说清 DP 的前提（完整模型）与局限（状态爆炸）。

## 核心概念

### 1. GPI（Generalized Policy Iteration，4.6）

```
   ┌──────────────┐  改进   ┌──────────────┐
   │  策略评估      │ ───────▶ │  策略改进      │
   │ (prediction)  │ ◀─────── │ (improvement) │
   └──────────────┘  评估    └──────────────┘
      两个方向都在逼近同一个最优解 (V*, π*)
```

DP = 有完整模型时，对 GPI 两腿"精确计算"的方法。

### 2. 策略评估（4.2，迭代版 Bellman）

给定 π，迭代：

```
V_{k+1}(s) = Σ_a π(a|s) [ R(s,a) + γ Σ_s' P(s'|s,a) V_k(s') ]
```

从 V_0=0 开始，直到 `max_s |V_{k+1}(s) − V_k(s)| < θ`（误差收缩性质：每步误差最多放大 γ 倍，所以一定收敛）。

### 3. 策略改进（4.3）

```
π'(s) = argmax_a [ R(s,a) + γ Σ_s' P(s'|s,a) V_π(s') ]
```

改进策略定理：若 V_π 已精确，则 π' 在每个状态都不差于 π（除非 π 已最优）。

### 4. 两种迭代（4.4 vs 4.5）

| | policy iteration | value iteration |
|---|---|---|
| 内层 | **完整**策略评估（迭代到收敛） | **一步** Bellman 备份 |
| 改进 | 每轮做一次 | 每轮顺手做 |
| 特点 | 轮数少，每轮贵 | 轮数多，每轮便宜；常提前"够用" |

### 5. 4×4 gridworld 例子（4.7，书的经典）

目标格 +1、惩罚格 −1、其余 0；动作向上/下/左/右，80% 按指令走、20% 随机走错方向；撞墙留在原地。观察：最优策略如何"绕开惩罚"、"提前转向"。

## 动手步骤

1. 建 gridworld（建议 4×4，含 1 个奖励格 + 1 个惩罚格 + 随机转移）：

```python
import numpy as np

W, H, G = 4, 4, 0.9
def transition(s, a):
    # 返回 [(s', r, p), ...]; 80% 直行 + 20% 侧偏, 撞墙 r=0 原地
    ...

def policy_evaluation(pi, V=None, theta=1e-8):
    V = np.zeros(W*H) if V is None else V.copy()
    while True:
        delta = 0
        for s in range(W*H):
            v = 0
            for a, pa in pi[s].items():
                for sp, r, p in transition(s, a):
                    v += pa * p * (r + G * V[sp])
            delta = max(delta, abs(v - V[s]))
            V[s] = v
        if delta < theta: break
    return V

def improve(V):
    pi = {}
    for s in range(W*H):
        q = {a: sum(p*(r+G*V[sp]) for sp,r,p in transition(s,a)) for a in range(4)}
        pi[s] = {a: 1.0 for a in q if q[a] == max(q.values())}   # 最优可能多个
    return pi

V, pi = np.zeros(W*H), None
while True:
    pi_new = improve(V)
    if all(pi_new[s] == pi[s] for s in range(W*H)) if pi else True:
        break
    V = policy_evaluation(pi_new)
    pi = pi_new
```

2. 输出：value map（`matplotlib imshow`）、策略箭头图；对比 policy iteration vs value iteration 的"轮数 × 每轮计算量"。
3. 记录 θ 的影响（1e-2 vs 1e-8）。

## 常见坑

- **改进时平局处理不一致** → 策略震荡、不收敛；固定 tie-break 规则（如动作编号小的优先）。
- 策略评估提前停（θ 太大）→ "改进策略定理"要求 V 足够精确，θ 太大可能来回震荡（书 4.4 讨论）。
- 转移概率写反（P(s'|s,a) vs P(s,a|s')）→ 价值图"镜像错乱"。
- γ 写成 0.9 但 Bellman 里漏乘 → 策略变"短视"，直接撞向最近奖励而不管路径。
- 把 DP 用到没有模型的环境 → DP 的前提是**精确知道 P 和 R**，不知道就用 MC/TD（W5–W6）。

## 自测

1. GPI 的"两腿"分别是什么？为什么它们会收敛到同一个解？
2. 写出策略评估的迭代式 + 收敛条件。
3. policy iteration 与 value iteration 的 3 个区别？
4. 策略改进定理的前提是什么（V_π 要满足什么）？
5. DP 需要什么前提？什么时候必须放弃 DP 转 MC/TD？

### 参考答案

1. 策略评估（把当前策略的价值算准）+ 策略改进（按价值贪心换策略）；两腿交替且各自单调变好，共同逼近 (V*, π*)。
2. V ← T_π(V)：V(s)=Σ_a π(a|s)[R(s,a)+γΣ P(s'|s,a)V(s')]；直到 max|ΔV|<θ。
3. 内层完整 vs 一步备份；轮数少每轮贵 vs 轮数多每轮便宜；每轮都有改进 vs 每轮顺手改进。
4. V_π 是"精确"的（或足够精确）策略评估结果。
5. 精确的转移模型 P 与奖励 R（完全已知的环境）；环境未知/太大 → MC/TD（无需模型，靠采样）。

## 笔记清单（对照书本补写）

- [ ] 画 GPI 循环图 + 标注 DP/MC/TD 各在哪个位置
- [ ] 抄写 policy evaluation 迭代式 + 误差收缩的直觉解释
- [ ] 记录 gridworld 的 value map 截图 + 策略箭头图
- [ ] 对比表：policy iteration vs value iteration（轮数/单轮成本/θ 敏感度）
- [ ] 写"DP 的天花板"：状态数 × 动作数 的乘法，为什么真实机器人用不了
