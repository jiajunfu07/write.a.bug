# W07 · n-step Bootstrapping（课件）

> 对应阅读: Ch7 n-step Bootstrapping (PDF p.163–180)，重点 7.1/7.2/7.6 Tree Backup、7.8 统一视角
> 里程碑: n 扫描曲线（n=1,2,4,8,∞）
> 配套笔记: 建议 `Chapter-7-n-step-Bootstrapping.md`（待写）

## 学习目标

1. 理解 n-step 是 TD 与 MC 之间的"连续谱"；
2. 写出 n-step Sarsa 的更新式；
3. 理解 Tree Backup 解决的 off-policy 问题；
4. 做 n 扫描实验并解释"为什么中间某个 n 最好"。

## 核心概念

### 1. 谱系视角（7.0/7.7）

```
n=1 (TD)  ←——— n=2,3,... ———→  n=∞ (MC)
 bootstrap 重            bias 低 / 方差高
```

- **n-step return**：`G^{(n)}_t = R_{t+1} + ... + R_{t+n} + γ^n V(S_{t+n})`
- n 小：更新频繁、bias 大（bootstrap 误差大）；n 大：接近 MC，方差大、更新慢；
- **最优 n 往往在中间**：样本效率 vs 方差的权衡（7.4 的曲线）。

### 2. n-step Sarsa（7.2）

```
Q(S_t,A_t) ← Q(S_t,A_t) + α [ G^{(n)}_t - Q(S_t,A_t) ]
G^{(n)}_t = R_{t+1}+...+R_{t+n} + γ^n Q(S_{t+n}, A_{t+n})
```

（on-policy：下一动作链来自行为本身。）

### 3. Off-policy n-step 的麻烦 + Tree Backup（7.6）

- 问题：轨迹里行为策略 b 与目标 π 的分叉 → 朴素 IS 权重爆炸（同 Ch5）；
- **Tree Backup (TB(n))**：不做 IS，改用"策略期望递推"：
  `Z_t = R_{t+1} + γ Σ_a π(a|S_{t+1}) [ (1−π(a_t+1|S_{t+1}))... + Q(S_{t+1},a) ]`
  直觉：**在每个分叉点按目标策略"展开期望树"**，天然 off-policy 安全（λ 版本见 W12）。

### 4. n-step Q(λ)（7.8，* 节，二遍读）

统一框架：`Q^{(λ)}` 把 TD(λ) 与 Q-learning 插值——理解"λ 是 n-step 的指数加权版本"（W12 的主角）。

## 动手步骤

1. 在 CartPole（Gymnasium）或 W6 gridworld 上实现 n-step Sarsa（n 可配）；
2. **n 扫描**：n ∈ {1, 2, 4, 8, 32, ∞(MC)}，同一 seed 集，画"平均 test return（每 200 episode 取 5 局）"曲线；
3. 标注每个 n 的"达到 0.9×最优"的 episode 数 → 找最优 n；
4. （进阶）实现 TB(1) 并与 n-step Sarsa 在同一 off-policy 设置下对比稳定性。

## 常见坑

- n 超过 episode 长度 → 实际退化为 MC（return 里没有 bootstrap 项），曲线会突然变平/变差，别当成 bug。
- 更新延迟 n 步（用 replay buffer 存最后 n 步）实现时，索引越界要在 episode 边界特殊处理。
- n-step 的 γ 要乘 n 次（γ^n），漏写 → 长程 reward 被低估。
- 用 n-step 比较不同算法时，"每 episode 的更新次数"变了 → 对比要按 **wall-clock/step 数** 归一，而不是 episode 数。
- off-policy n-step 直接套 IS 权重 → 方差爆炸，曲线"抽风"（这正是 TB 的出场理由）。

## 自测

1. n-step return 的公式（带 bootstrap 项）？n=∞ 时它变成什么？
2. 为什么"最优 n 往往不是 1 也不是 ∞"？
3. Tree Backup 与朴素 IS 的本质区别？
4. n-step Sarsa 每 episode 更新多少次（episode 长度 T）？n=1 与 MC 各是多少？
5. 对比实验为什么应该按 step 数归一？

### 参考答案

1. G^{(n)}_t = R_{t+1}+...+R_{t+n} + γ^n V(S_{t+n})；n→∞ 时 bootstrap 项消失，成为 MC 的 n 步 return。
2. 小 n：更新快但 bootstrap bias 大；大 n：无 bias 但方差大且更新稀疏；中间 n 在偏差-方差权衡上样本效率最高（7.4）。
3. IS 用"实际走过的轨迹概率"做权重（连乘爆炸）；TB 在每个分叉点按目标策略期望展开（无连乘），结构上避免 off-policy 偏差。
4. 约 T−n 次；n=1 约 T 次；MC（n=∞）每 episode 每状态 1 次（first-visit）。
5. 不同 n 的更新频率不同，按 episode 数对比会把"更新次数"混进"算法好坏"里。

## 笔记清单（对照书本补写）

- [ ] 画 TD↔MC 谱系图（n 轴）+ 偏差/方差标注
- [ ] 抄写 n-step Sarsa 更新式 + γ^n 的作用
- [ ] 记录 n 扫描曲线 + "最优 n"结论（附 step 归一表）
- [ ] 用自己的话写 Tree Backup 的"期望树"直觉（画一棵 2 层小树）
- [ ] 整理 TD/MC/n-step 的"更新频率 × 单步信息量"对比表
