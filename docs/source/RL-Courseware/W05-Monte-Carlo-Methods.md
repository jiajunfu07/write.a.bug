# W05 · 蒙特卡洛方法（课件）

> 对应阅读: Ch5 Monte Carlo Methods (p.91–118)，重点 5.1–5.5
> 里程碑: 同一 gridworld 的 MC vs DP 叠图
> 配套笔记: 建议 `Chapter-5-Monte-Carlo-Methods.md`（待写）

## 学习目标

1. 掌握 MC prediction：用 episode 平均估计 V（first-visit / every-visit）；
2. 掌握 MC control：exploring starts 与 ε-soft（without ES）；
3. 理解 off-policy MC + importance sampling 的权重与爆炸问题；
4. 在同一任务上对比 MC 与 DP 的样本效率。

## 核心概念

### 1. MC prediction（5.1）

不需要模型，**必须等 episode 结束**，用实际 return 做平均：

```
V(s) ← V(s) + [1/N(s)] * (G(s) - V(s))        # incremental, 同 W2 老虎机
```

- **first-visit**：一个 episode 里同一状态只计第一次；
- **every-visit**：每次出现都计；
- 为什么必须等结束？G_t 是"未来所有 reward 的和"，提前平均等于引入 bias（MC 的无偏性依赖完整 return）。

### 2. MC control（5.2–5.4）

- **exploring starts (ES)**：初始状态按某种概率覆盖所有状态 → 保证每个 (s,a) 都能被采到；
- **without ES**：策略必须**永远探索** → 用 **ε-soft**（每个动作概率 ≥ ε/|A|）；
- 循环：collect episodes → 更新 Q → `π(s)=argmax Q`（保留 ε 探索）。

### 3. Off-policy MC + Importance Sampling（5.5）

- 行为策略 b（探索用）、目标策略 π（想评估的）；
- 整条轨迹的 IS 权重：

```
ρ = [π(a|s)/b(a|s)] 的连乘, 从 s 到 episode 结束
Q(s,a) ← Q(s,a) + [1/N] * ρ * (G - Q)
```

- **问题**：ρ 是长连乘，数值极易爆炸/趋零 → 方差巨大（Ch11 "死亡三联症"的前奏）。

### 4. MC vs DP vs TD（建立坐标系）

| | 需要模型 | 更新时机 | 目标 |
|---|---|---|---|
| DP | 要（精确 P,R） | 任意时（用 P 算期望） | 预测/控制 |
| MC | 不要 | episode 结束 | 预测/控制 |
| TD | 不要 | 每一步（bootstrap） | 预测/控制（W6） |

## 动手步骤

1. 复用 W4 的 gridworld（有模型，所以同时能跑 DP 与 MC，公平对比）；
2. **MC control（ε-soft）**：

```python
def mc_control(episodes=300, eps=0.1, gamma=0.9):
    Q = {}; N = {}
    for ep in range(episodes):
        s = start(); done = False; rets = {}
        while not done:
            a = eps_greedy(Q, s, eps, init=1.0)
            sp, r, done = step(s, a)
            # 记录 first-visit return
            if s not in rets: rets[s] = []
            ...
        # episode 结束后更新所有 first-visit 的 (s,a)
        ...
    return Q
```

3. 画对比图：DP 的 V*（W4）vs MC 收敛后的 V（同一任务）；记录"MC 需要多少 episode 才接近 DP"。
4. 变体实验：ε=0.05/0.1/0.2；exploring starts vs ε-soft 的曲线差异。

## 常见坑

- 更新发生在 episode 中间（提前平均）→ 引入 bias，MC 无偏性被破坏。
- first-visit 与 every-visit 混用 → 曲线不可解释（先固定 first-visit）。
- IS 权重没截断（clipping）→ 个别轨迹权重 1e8，曲线抖动；可试 weighted IS + 截断对比。
- ε-soft 的 π 与 Q 更新后不一致（忘了同步改进策略）→ 收敛到非最优。
- 用"随机初始状态"冒充 exploring starts → 某些 (s,a) 永远采不到（reachable 问题）。

## 自测

1. MC prediction 为什么必须等 episode 结束？
2. first-visit 与 every-visit 的区别？
3. 没有 exploring starts 时，MC control 的策略必须满足什么条件？为什么？
4. 写出 off-policy MC 的 IS 更新式；ρ 连乘为什么危险？
5. MC 与 DP 在"需要模型"上的差异导致什么实际差别？

### 参考答案

1. G_t 依赖未来全部 reward；提前平均会把"尚未发生的 reward"当已知，估计有偏（MC 的无偏性 = 完整 return 的期望）。
2. 一个 episode 内同一状态只取第一次出现 / 每次出现都计入。
3. 每个 (s,a) 必须被无限次访问 → 策略对每个动作的概率下界 >0（ε-soft），否则某些 Q 永远无样本。
4. Q ← Q + (1/N)·ρ·(G−Q)，ρ=Π π(b/a)/b(b/a)；长连乘数值爆炸/趋零，方差随轨迹长度指数增长。
5. DP 一步算精确期望（要模型）；MC 靠大量采样逼近（无模型但样本效率低）。

## 笔记清单（对照书本补写）

- [ ] 抄写 MC prediction 的 incremental update + first/every-visit 定义
- [ ] 画 "collect → update Q → improve π" 的 MC control 循环图
- [ ] 记录 MC vs DP 对比图（同一 gridworld）+ 样本效率结论
- [ ] 用自己的话解释 IS 权重爆炸（配一个数值小例子）
- [ ] 整理 MC/DP/TD 三行对比表（为 W6 的 TD 留一格待填）
