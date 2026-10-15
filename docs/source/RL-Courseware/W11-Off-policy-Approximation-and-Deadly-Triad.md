# W11 · off-policy 近似与"死亡三联症"（课件）

> 对应阅读: Ch11 *Off-policy Methods with Approximation (PDF p.279–308)，重点 11.2 发散例、11.3 deadly triad、11.5 Bellman error
> 里程碑: 发散复现实验 + 解释报告
> 配套笔记: 建议 `Chapter-11-Deadly-Triad.md`（待写）

## 学习目标

1. 复现"线性 + bootstrapping + off-policy"的发散；
2. 说清 deadly triad 三件各是什么、为什么"齐上"才危险；
3. 理解 Bellman error 的几何（11.6）与"它不可学习"的结论（11.7）；
4. 把 DQN 的两剂解药放回这个框架里解释。

## 核心概念

### 1. 三件"武器"（11.0/11.3）

| 件 | 内容 |
|---|---|
| ① 函数近似 | V(s;w)=wᵀφ(s)，参数化 + 外推 |
| ② Bootstrapping | 用当前估计当 target（TD/Q-learning） |
| ③ Off-policy | 行为策略 b ≠ 目标 π（如用 ε-greedy 学贪心） |

**单独或两件**：大多收敛（on-policy + 近似 + bootstrap 在多数设定下 OK；off-policy + 表格 + bootstrap = Q-learning 也 OK）。
**三件齐上**：存在收敛失败的例子（11.2）——这就是"死亡三联症"。

### 2. 发散例（11.2，要能画出）

经典构造：2 状态、2 动作、线性 1 特征，行为策略固定、目标贪心 → 每步"argmax 跳来跳去"，w 沿对角线飞出去，loss 看似在降、值却爆炸。
关键机制：**argmax 在边界翻转** → target 不连续 → semi-gradient 追一个跳动的目标 → 正反馈。

### 3. Bellman error 的几何（11.6）

- 把所有 (s,a) 的 Bellman error 看成空间里的点，"可行解"是一个（通常空）交集区域；
- 线性近似的"最优 w" = 到该区域最近的点（最小化期望 Bellman error 平方）；
- **11.7 结论**：Bellman error 本身"不可学习"（stochastic approximation 对它不收敛），但它的**期望平方**可以 → 引出 Gradient-TD（11.8）。

### 4. Gradient-TD / Emphatic-TD（11.8–11.9，二遍读）

- Gradient-TD：对"期望 Bellman error 平方"求真正的梯度 → 理论上收敛；
- Emphatic-TD：加 emphatic 系数降方差 → 实践中更接近 semi-gradient 的速度。
- 实践启示：DQN 的"工程稳定"（replay + target net）是绕开这套理论问题的实用路径。

## 动手步骤

1. **复现 11.2 发散**（~40 行，线性 1 特征 off-policy TD）：

```python
import numpy as np
# 按书 11.2 的 2x2 例子: s1/s2, a1/a2, 固定行为策略, 目标=贪心
w = np.zeros(1)          # V(s) = w * phi(s), phi 按书定义
alpha = 0.5
for t in range(1000):
    s, a = sample_behavior(t)          # 固定行为策略
    r, sp = env_step(s, a)
    a_greedy = argmax_greedy(w, sp)    # 用当前 w 选贪心
    target = r + gamma * w.dot(phi(sp, a_greedy))   # off-policy bootstrap
    delta = target - w.dot(phi(s, a))
    w = w + alpha * delta * phi(s, a)
    if t % 10 == 0: print(t, w)
```

   观察：w 指数级增长（loss 曲线先降后爆炸）。
2. **对照实验**（证明是"三件齐上"的锅）：
   - 去掉 off-policy（行为=贪心）→ 不炸；
   - 去掉 bootstrapping（用 MC return）→ 不炸；
   - 去掉近似（表格 V）→ 不炸；
3. 写 1 页报告：发散曲线 ×4 + "哪一件是导火索"的讨论（提示：argmax 翻转）。

## 常见坑

- 复现时随机性太强 → 发散被"噪声"掩盖；按书 11.2 用**确定性行为策略**才能稳定复现。
- 把"loss 上升"当发散判据 → 先看 **w 的范数**（‖w‖ 指数增长才是真发散）。
- 用 2 特征以上复现 → 现象更乱，先做书的 1 特征最小例。
- 报告里把"工程上 DQN 能跑"当成"理论收敛" → 两者是不同命题（实践 vs 证明）。
- 忘记 γ：11.2 例子 γ 取 0.9 左右才炸；γ=0 退化成 bandit，不炸。

## 自测

1. 死亡三联症的三件 + "哪两件齐上通常没事"？
2. 11.2 发散例的"导火索"是什么机制？
3. 为什么表格法 Q-learning（off-policy+bootstrap）不发散，而线性版会？
4. DQN 的 replay + target net 分别对应"缓解三件中的哪一件"？
5. Gradient-TD 与 semi-gradient 的根本区别？

### 参考答案

1. 函数近似、bootstrapping、off-policy；任意两件（近似+bootstrap 的 on-policy、off-policy+bootstrap 的表格）多数设定收敛。
2. argmax 在决策边界翻转 → target 不连续跳变 → semi-gradient 正反馈放大 w。
3. 表格的参数=状态数，更新只动一个格子（局部）；线性近似的参数被所有状态共享（外推），一个格子的错误更新会"辐射"到全局，放大跳动。
4. 都主要缓解"off-policy + bootstrap"的组合：replay 降低样本相关性/方差；target net 固定化 target，抑制正反馈（不消除近似，也不消除 off-policy 本身）。
5. semi-gradient 对"移动的 target"直接做近似更新（不保证收敛）；Gradient-TD 对"期望 Bellman error 平方"求真梯度（有收敛保证，代价是方差/计算）。

## 笔记清单（对照书本补写）

- [ ] 画 11.2 发散例的状态-动作图 + w 轨迹（t vs ‖w‖）
- [ ] 记录 4 组对照实验曲线（全炸 / 去 off-policy / 去 bootstrap / 去近似）
- [ ] 抄写 Bellman error 最小化目标 + "不可学习"结论
- [ ] 用自己的话写"为什么 DQN 是工程解不是理论解"
- [ ] 整理"稳定化手段清单"（replay/target net/clip/双网络）+ 各自针对的机制
