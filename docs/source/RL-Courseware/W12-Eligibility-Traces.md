# W12 · 资格迹：TD(λ)（课件）

> 对应阅读: Ch12 Eligibility Traces (p.287–320)，重点 12.1 λ-return、12.2 TD(λ)、12.4 True Online、12.7 Sarsa(λ)
> 里程碑: λ 扫描曲线（0 → 1）+ 稳定性结论
> 配套笔记: 建议 `Chapter-12-Eligibility-Traces.md`（待写）

## 学习目标

1. 掌握 λ-return 的递归定义，理解它是"n-step 的指数插值"；
2. 实现 TD(λ)（forward-view 与 replacing/dumping 两种 trace 更新）；
3. 理解 eligibility trace 的功劳分配直觉；
4. 做 λ 扫描，解释"λ 不是越大越好"。

## 核心概念

### 1. λ-return（12.1，核心公式）

把 n-step return 按 (1−λ)λ^{n−1} 加权：

```
G_t(λ) = (1-λ) Σ_{n=1..∞} λ^{n-1} G_t^{(n)}
       = R_{t+1} + γ[(1-λ)V(S_{t+1}) + λ G_{t+1}(λ)]      (递归, 实用)
```

- λ=0 → G^{(1)}（TD(0)）；λ=1 → G^{(∞)}（MC return）；
- **λ-return = 在"偏差(小 n)"与"方差(大 n)"之间的平滑插值**。

### 2. TD(λ)（12.2）

每步对所有"近期访问过的状态"做更新，按 trace 加权：

```
e_t(s) = λ e_{t-1}(s) * [1(s == S_{t-1})]     (replacing)
       或 e_t(s) = λ e_{t-1}(s) + 1(s == S_{t-1})  (incrementing)
V(s) ← V(s) + α δ_t e_t(s)      对所有 e_t(s) ≠ 0
δ_t = R_{t+1} + γ V(S_{t+1}) - V(S_t)
```

### 3. 直觉（必写进笔记）

> **eligibility trace = "最近访问过的状态/特征的衰减记忆"，功劳（或锅）按最近程度分配。**

例：走了 10 个状态后撞墙 −1，trace 让"撞墙前 3 步"的状态多担责、更早的状态少担责——比 TD(0) 只怪"上一步"公平得多。

### 4. True Online TD(λ)（12.4）

naive 的 TD(λ) 在"同一状态重复访问"时更新不一致（取决于访问顺序）→ True Online 版用 δ 的递归定义，保证**与访问顺序无关**。

### 5. Sarsa(λ) 与 Q(λ)↔TB(λ)（12.7–12.10）

- Sarsa(λ)：把 trace 加到 (s,a) 对上（on-policy 控制版）；
- **Q(λ) vs TB(λ) 谱系**（12.10）：off-policy trace 方法的两个端点：
  - Q(λ)（Watkins）：快但可能不稳定（off-policy + trace 的方差）；
  - TB(λ)（Tree Backup）：稳但慢（每步按 π 期望展开）；
  - 中间：`Q(λ)↔TB(λ)` 连续谱（书 12.10 的统一视角）——**这一章是整本书"off-policy 稳定性"的总收尾**。

## 动手步骤

1. **λ 扫描（CartPole 或 MountainCar，on-policy Sarsa(λ) 或 TD(λ) 预测）**：

```python
def td_lambda_step(V, e, s, a, r, sp, done, alpha, gamma, lam):
    # forward-view TD(lambda), replacing traces
    delta = r + gamma * (0 if done else V.get(sp, 0)) - V.get(s, 0)
    e[s] = 1.0
    for st in list(e):
        V[st] = V.get(st, 0) + alpha * delta * e[st]
        e[st] *= gamma * lam
        if e[st] < 1e-8: del e[st]
```

2. λ ∈ {0, 0.25, 0.5, 0.75, 0.9, 1.0}，同一 seed 集，画"test return（滑动 5 局均值）"曲线；
3. 标注每个 λ 达到 0.9×最优的 episode 数 → 找最优 λ；
4. （进阶）对比 replacing vs incrementing trace 的差异曲线。

## 常见坑

- **trace 衰减忘乘 γ**（`e *= λ` 而不是 `γλ`）→ 长 trace 权重虚高，曲线怪；
- trace 字典无限膨胀 → 设阈值删除（<1e-8）或定期清理；
- λ=1 时 trace 永不衰减 → 退化成"全局平均更新"，CartPole 后期反而抖；
- 把 λ-return 当成"再跑一遍 MC" → 它是**递归**算的（每步 O(1)），别每步都展开 Σ_n；
- 对比实验时 α 没随 λ 调整 → 有效步长变了，结论偏（固定"总学习量"再比）。
- Sarsa(λ) 的 trace key 是 (s,a)，别沿用 TD(λ) 的 s-only trace。

## 自测

1. 写出 λ-return 的递归式；λ=0/1 各退化为什么？
2. eligibility trace 的直觉（一句话 + 一个撞墙例子）？
3. replacing vs incrementing trace 的区别？
4. 为什么 λ 不是越大越好？
5. Q(λ) 与 TB(λ) 分别偏向"快"还是"稳"？为什么？

### 参考答案

1. G_t(λ)=R_{t+1}+γ[(1−λ)V(S_{t+1})+λG_{t+1}(λ)]；λ=0→TD(0) 的 n=1 return；λ=1→完整 MC return。
2. 按"最近访问程度"给状态分功劳/锅；撞墙前 3 步 trace 还热、多担责，10 步前的已衰减、少担责。
3. replacing 每次访问把 trace 重置为 1；incrementing 累加（重复访问 trace 更大）。
4. λ 大→方差大（接近 MC）+ 更新更"全局"（局部性差），小 λ 偏差大；最优在中间（样本效率峰值）。
5. Q(λ) 快（直接 max/bootstrap 实际轨迹）但 off-policy 方差不稳；TB(λ) 稳（按 π 期望展开，无 IS 爆炸）但更新保守、收敛慢。

## 笔记清单（对照书本补写）

- [ ] 抄写 λ-return 递归式 + TD(λ) 更新式（含 trace 更新）
- [ ] 画"trace 衰减"示意图（状态序列 + e 值柱状）
- [ ] 记录 λ 扫描曲线 + 最优 λ + α 归一说明
- [ ] 用自己的话写"功劳分配"直觉（3 句以内）
- [ ] 整理 Q(λ)/TB(λ)/interpolated 三行对比表（快/稳/位置）
