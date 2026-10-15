# W14 · 心理学与神经科学（课件）

> 对应阅读: Ch14 Psychology (PDF p.363–398) 重点 14.3；Ch15 Neuroscience (PDF p.399–442) 重点 15.4–15.6
> 里程碑: Rescorla–Wagner blocking 模拟 + TD 延迟强化 RPE 波形
> 配套笔记: 建议 `Chapter-14-15-Brain-and-RL.md`（待写）

## 学习目标

1. 掌握 Rescorla–Wagner 模型并复现 blocking 效应；
2. 理解 TD 模型如何"预测"延迟强化的 RPE 波形；
3. 说清多巴胺 reward-prediction-error 假说的核心证据；
4. 能画出"TD error ↔  dopamine"的对应图。

## 核心概念

### 1. Rescorla–Wagner（14.3.2，条件反射的学习率方程）

```
ΔV_i = α_i β_i (λ - Σ_j V_j) * x_i
```

- x_i：刺激 i 是否呈现（0/1）；V_i：该刺激的条件反应强度；λ：US（非条件强化）强度；
- **预测误差驱动学习**：`(λ − ΣV)` = "实际 US − 预期 US" —— 与 TD error 同构！

### 2. Blocking 效应（14.3.1，经典实验）

1. 阶段一：A→US 反复配对 → V_A 学到 λ；
2. 阶段二：AB→US 配对 → B 几乎学不到（"被 A 挡住"）；
- 原因：阶段二开始时 ΣV≈λ → 预测误差≈0 → B 的 ΔV_B≈0；
- **RL 视角**：这是"预测误差驱动学习"在心理学里的第一个铁证。

### 3. TD 模型（14.3.3）+ 延迟强化（14.4）

- 把 RW 的"误差项"换成时间维度的 TD error：`δ_t = r_t + γV(s_{t+1}) − V(s_t)`；
- **预测**：如果强化延迟 T 秒，RPE 信号应该：
  - 强化出现瞬间 +1 尖峰（首次）；
  - 随着学习，正尖峰**逐渐前移**到"强化前 T 秒"；
  - 强化出现瞬间反而变成**负尖峰**（期待落空）。
- 这正是 Woldroyt/Schultz 等电生理实验观察到的多巴胺波形（15.6）——理论与实验的闭环。

### 4. 多巴胺 RPE 假说（15.4–15.5）

- 中脑多巴胺神经元（SNc/VTA）对**意外** reward 放电增强、对**预期内** reward 放电减弱、对惩罚/期待落空放电抑制；
- "dopamine = 教学信号（teaching signal）"的假说 + 争议（hedonistic vs teaching，15.8 的讨论）；
- 神经 actor-critic（15.7）：actor（运动皮层/基底节直通路）↔ critic（纹状体/黑质）的对应——**脑里可能真有一套 actor-critic**。

### 5. 为什么这一章值得读（课程定位）

RL 不是"纯工程"：它的核心方程（TD error、prediction error）在**学习理论**（RW）、**神经科学**（dopamine RPE）、**AI**（TD-Gammon/AlphaGo）三条线里反复出现。W14 = 把三条线拧到一起。

## 动手步骤

1. **Blocking 模拟（RW 模型，~30 行）**：

```python
import numpy as np, matplotlib.pyplot as plt

alpha, beta, lam, T = 0.1, 1.0, 1.0, 60
V = np.zeros(2)          # [V_A, V_B]
xA, xB, hist = 0, 0, []

# Phase 1: A -> US
for t in range(T):
    dV = np.array([alpha*beta*(lam - V.sum())*1, 0])   # 只有 A
    V += dV
# Phase 2: A+B -> US
for t in range(T):
    V += np.array([alpha*beta*(lam - V.sum()), alpha*beta*(lam - V.sum())])
    hist.append(V.copy())

hist = np.array(hist)
plt.plot(hist[:,0], label="V_A"); plt.plot(hist[:,1], label="V_B")
plt.axvline(0, ls="--"); plt.legend(); plt.show()
```

   验收：阶段二 V_B 几乎不涨（blocking 成立）；做对照"先 B 后 AB" → 这次 A 被 block。
2. **TD 延迟强化 RPE 波形**：一维"到达目标"任务（状态=位置 0..10，目标 10 给 r=1，γ=0.9）；
   - 跑 200 个 episode 的 TD(0)（α=0.3）；
   - 每个 episode 记录 δ_t；画"前 5 个 + 后 5 个 episode 的 δ 沿位置图"→ 观察正峰从位置 10 前移到 9；
3. 对照文献截图（Schultz 1997 的 dopamine 响应图，W15 读 15.5 时找）。

## 常见坑

- RW 的 α/β 没分离（两个刺激用不同 α）→ blocking 不对称，结论乱；先 α_A=α_B 做对称版。
- 阶段二"AB 同时呈现"写成"先 A 后 B 交替" → 不是 blocking 实验（顺序条件化是另一个现象，14.3.1）。
- TD 延迟强化实验 γ 设 1.0 → 波形前移不明显（γ<1 才有"时间折扣"的清晰前移）。
- 把"多巴胺 = 快乐"当结论 → 书 15.8 明确讨论 hedonistic 与 teaching 两种假说及证据边界。
- 模拟曲线"太干净" → 加一点噪声/有限样本会更像实验数据（讨论时加分）。

## 自测

1. Rescorla–Wagner 方程里哪一项是"预测误差"？
2. blocking 效应的两阶段实验设计 + RL 视角解释。
3. 延迟强化下，RPE 正峰如何随学习移动？为什么？
4. 多巴胺 RPE 假说的三条核心证据？
5. "神经 actor-critic"里 actor 与 critic 各对应什么脑区（书 15.7 的对应）？

### 参考答案

1. `(λ − Σ_j V_j)`：实际 US 强度减去条件刺激们已建立的预期。
2. 阶段一 A→US 学到 V_A≈λ；阶段二 AB→US 时预期已满足（ΣV≈λ），误差≈0 → B 学不到。RL 视角：学习由预测误差驱动，A 已"解释"了 US。
3. 正峰从"强化时刻"逐渐前移到"强化前约 −log(γ) 步"；因为 V(s) 学到"未来会有 r"，δ 在"预期突然升高"处最大（前移的起点）。
4. 意外 reward → 多巴胺爆发；预期内 reward → 爆发减弱/消失；期待落空/惩罚 → 放电抑制（与 TD error 符号对应）。
5. critic ≈ 黑质/纹状体（价值与误差估计）；actor ≈ 运动皮层 + 基底节直通路（动作选择/执行）——对应见 15.7。

## 笔记清单（对照书本补写）

- [ ] 抄写 RW 方程 + TD error 的同构对照表
- [ ] 画 blocking 两阶段曲线（含"先 B 后 AB"对照组）
- [ ] 记录延迟强化 δ 波形图（前 5 vs 后 5 episode）
- [ ] 用自己的话写"为什么 RL 的方程在脑里也出现"（3–5 句）
- [ ] 整理 dopamine 证据清单（支持/反对 teaching 假说各 2 条）
