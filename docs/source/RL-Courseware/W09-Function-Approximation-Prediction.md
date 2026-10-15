# W09 · 函数近似：预测（课件）

> 对应阅读: Ch9 On-policy Prediction with Approximation (PDF p.219–264)，重点 9.3/9.4/9.5 tile coding/9.7 ANN
> 里程碑: MountainCar 的线性(tile) + 小网络 TD 预测
> 配套笔记: 建议 `Chapter-9-Function-Approximation.md`（待写）

## 学习目标

1. 理解从"表格"到"参数化函数"的动机（状态爆炸 + 泛化）；
2. 掌握线性近似 + stochastic/semi-gradient 更新；
3. 会做 tile coding，说清它为什么能外推；
4. 用一个小神经网络做 TD 预测，理解 LSTD 的存在意义。

## 核心概念

### 1. 为什么要近似（9.0）

- 表格法的两个天花板：状态数 × 参数数爆炸；无泛化（相邻状态值可以天差地别）；
- **函数近似**：V(s;w) = wᵀφ(s)，φ 是特征向量 → 参数共享 → 泛化。

### 2. Semi-gradient（9.3，术语必懂）

目标本身就是 w 的函数（TD target 含 V(s';w)）→ 不是真正的梯度下降，叫 **semi-gradient**：

```
w ← w + α [ δ φ(s) ],   δ = r + γ V(s';w) - V(s;w)
```

（对比：线性回归是"真梯度"，因为目标 y 是固定的。）

### 3. 线性方法（9.4）

`V(s;w) = Σ_k w_k φ_k(s)`；特征可以是：
- 独热（= 表格，无泛化）
- 多项式、Fourier（9.5.1–2）
- **tile coding / 粗编码**（9.5.3–4）：把状态空间切成重叠的"瓦片"，每个瓦片一个特征 → 局部线性 + 平滑过渡；
- RBF（9.5.5）。

**tile coding 的直觉**：像"地图分格但每格有重叠"——位置稍微移动，激活的特征平滑变化 → 学到的价值面是连续的。

### 4. 神经网络（9.7）

- 特征 φ 换成隐层非线性：V(s;w) = MLP(s)；
- 更新仍是 semi-gradient（目标动 → 用当前 target 当常数做梯度）；
- 这是后面 DQN / policy gradient 的"地基"。

### 5. LSTD（9.9，概念层）

- semi-gradient 是"近似解一个移动的方程"，**不保证收敛**（Ch11 的发散根源）；
- **LSTD**：解"最小化 Bellman error 的期望"的正规方程，天然收敛（但大规模时解矩阵贵 → 引出 Ch11 的 Gradient-TD）。

## 动手步骤

1. **MountainCar（Gymnasium `MountainCar-v0`）**：state=(pos,vel) 连续，表格法不可能 → 天然的近似法实验场。
2. **线性 TD 预测（tile coding 手写版）**：

```python
import numpy as np

def tiles(pos, vel, n_pos=10, n_vel=8, overlap=2):
    i = int(np.clip((pos+1.2)/(2.4), 0, 1) * (n_pos-1+overlap))
    j = int(np.clip((vel+0.07)/0.14, 0, 1) * (n_vel-1+overlap))
    feats = set()
    for di in range(overlap):
        for dj in range(overlap):
            feats.add((i+di, j+dj))
    return sorted(feets) if False else sorted(feats)   # 重叠瓦片特征

w = {}   # tile -> value
alpha, gamma = 0.1, 1.0
s, _ = env.reset()
while ...:
    ts = tiles(*s)
    V_s  = sum(w.get(t, 0.0) for t in ts) / len(ts)
    a = ...; sp, r, term, trunc, _ = env.step(a)
    tsp = tiles(*sp)
    V_sp = sum(w.get(t, 0.0) for t in tsp) / len(tsp)
    delta = r + gamma*(0 if (term or trunc) else V_sp) - V_s
    for t in ts:  w[t] = w.get(t, 0.0) + alpha*delta
    for t in tsp: w[t] = w.get(t, 0.0) + alpha*delta   # (on-policy 预测简化)
```

3. **小 MLP 版**（PyTorch，~20 行）：`Linear(2,16)-ReLU-Linear(16,1)`，损失 = TD error 的平方（semi-gradient）；
4. 对比：线性 tile vs MLP 的"平均 episode reward 曲线" + 学到的 V(pos,vel) 热力图（2D 网格采样 V，画 imshow）。

## 常见坑

- **γ=1 + 无限 episode**：MountainCar 默认 γ 处理要显式（Gymnasium 内部折扣），自己算 return 时别混。
- tile 编码的越界（pos/vel 超出范围）→ clip 到 [0,1] 再切瓦片（代码里已 clip）。
- 学习率 α 对"每个特征"生效 → 重叠瓦片多，等效步长变大；α 从 0.05 起步扫。
- MLP 版忘了 `detach` target（`V_sp = ...; loss = (r + γ*V_sp.detach() - V_s)**2`）→ 反向传播穿过移动目标，训练飘。
- 用"同一张表"对比线性与 MLP 的公平性：同 seed、同 episode 数、同 γ。
- 把"预测任务"当"控制任务"做（没定义策略）→ 预测只需要给定行为策略下收集数据（随机策略即可）。

## 自测

1. 表格法 → 函数近似解决了哪两个问题？
2. 什么是 semi-gradient？和普通梯度下降的区别？
3. tile coding 为什么能提供"泛化"？（画两个重叠瓦片）
4. LSTD 与 semi-gradient 的区别？
5. 为什么 DQN 的训练里 target 要 detach / 用慢网络？（预告 W10）

### 参考答案

1. 参数爆炸（w 数量与状态数解耦）+ 泛化（相邻状态共享参数）。
2. 更新目标本身依赖被估计的参数（TD target 含 V(s';w)），不是对固定目标的梯度下降。
3. 状态空间被重叠瓦片覆盖，值函数 = 瓦片值平均 → 空间上连续、平滑，未见过的位置也能由邻近瓦片插值。
4. semi-gradient 每步近似解一个"移动的方程"（不保证收敛）；LSTD 解期望 Bellman error 最小的正规方程（收敛保证，但计算贵）。
5. target 若随 w 一起反传，等价于解一个自指的、可能病态的方程（死亡三联症前奏）；detach/慢网络让 target 近似固定 → 稳定。

## 笔记清单（对照书本补写）

- [ ] 抄写 semi-gradient 更新式 + 与真梯度下降的对比
- [ ] 画 tile coding 示意图（2D 状态 + 重叠瓦片 + 查询过程）
- [ ] 记录 MountainCar 线性 vs MLP 曲线 + V 热力图
- [ ] 用自己的话写"泛化 = 参数共享 = 平滑假设"
- [ ] 整理"特征选择"清单（独热/多项式/Fourier/tile/RBF/MLP）+ 各自适用
