# W13 · 策略梯度 + 项目二（课件）

> 对应阅读: Ch13 Policy Gradient Methods (PDF p.343–360)，重点 13.2 PG theorem、13.3 REINFORCE、13.4 baseline、13.5 actor-critic
> 里程碑: **项目二 · 深度 RL 实验** + 阶段考 3
> 配套笔记: 建议 `Chapter-13-Policy-Gradient.md`（待写）

## 学习目标

1. 理解"直接学策略"的动机（连续动作、随机策略、无需逆推动作）；
2. 推导（给提示链）policy gradient theorem，掌握 REINFORCE 更新式；
3. 说清 baseline 为什么降方差且保持无偏；
4. 用 SB3 完成 DQN vs PPO 对比实验，交付项目二。

## 核心概念

### 1. 为什么直接学 π（13.1）

- 价值方法最终要 `a = argmax Q`：连续动作空间里 argmax 不好做，且策略被"硬投影"成确定性的；
- **policy gradient**：参数化 π_θ(a|s)，直接对 θ 做梯度上升 —— 策略天然是（随机的）、可处理连续动作。

### 2. 目标函数与 PG theorem（13.2）

```
J(θ) = E_π[ G_0 ] = Σ_s μ_π(s) Σ_a π_θ(a|s) Q^π(s,a)

∇_θ J(θ) = E_π[ Σ_t ∇_θ log π_θ(A_t|S_t) * Q^π(S_t, A_t) ]        (PG theorem, 无模型)
```

**推导提示链**：对 log π 求导（log-derivative trick：∇log p = ∇p/p）→ 把"策略出现的概率 × 梯度"合并 → 期望可以采样估计。

### 3. REINFORCE（13.3）

用蒙特卡洛 return 代替 Q（无 bias）：

```
∇_θ J ≈ (1/T) Σ_t ∇_θ log π_θ(A_t|S_t) * G_t
```

- **高方差**（return 噪声大）→ 引入 **baseline**（13.4）：

```
∇_θ J ≈ Σ_t ∇_θ log π_θ(A_t|S_t) * (G_t - b(S_t))
```

  b(S) 只依赖状态（不含 a）→ 期望为 0 → **无偏不变**，但方差大降（常取 b=V(s) → 变成 advantage A(s,a)=Q−V）。

### 4. Actor-Critic（13.5）

- **Actor** = 策略 π_θ（选动作）；**Critic** = 价值 V_φ（估 baseline/advantage）；
- advantage 估计 `Â ≈ r + γV(s') − V(s)`（TD 风格）→ 方差比 MC return 小；
- 这就是 PPO/A2C/DDPG 家族的骨架（书外补充，Spinning Up 有完整实现）。

### 5. 连续动作的参数化（13.6）

高斯策略：π(a|s) = N(μ_θ(s), σ²)（σ 可学或固定）→ `log π = −(a−μ)²/2σ² + c` → 梯度干净可算。→ PPO 的默认做法。

## 动手步骤

1. **REINFORCE 手写版**（CartPole-v1，~40 行）：

```python
import torch, torch.nn as nn, gymnasium as np_env, numpy as np

class Policy(nn.Module):
    def __init__(s_, n_in, n_act):
        super().__init__()
        s_.net = nn.Sequential(nn.Linear(n_in, 64), nn.Tanh(), nn.Linear(64, n_act))
    def forward(s_, x): return torch.softmax(s_.net(x), dim=-1)

env = gym.make("CartPole-v1")
pol = Policy(env.observation_space.shape[0], env.action_space.n)
opt = torch.optim.Adam(pol.parameters(), 1e-3)

for ep in range(500):
    s, _ = env.reset(seed=ep); logps, rews = [], []
    done = False
    while not done:
        p = pol(torch.tensor(s, dtype=torch.float32))
        a = int(torch.distributions.Categorical(p).sample())
        logps.append(p[a].log()); s, r, terminated, truncated, _ = env.step(a)
        rews.append(r); done = terminated or truncated
    G, updates = 0.0, []
    for lp, r in zip(reversed(logps), reversed(rews)):
        G = r + 0.99 * G
        updates.append(lp * G)
    loss = -torch.stack(updates).mean()
    opt.zero_grad(); loss.backward(); opt.step()
    if ep % 50 == 0: print(ep, sum(rews))
```

   （进阶：把 G 换成 `r + γV(s') − V(s)` 的 advantage → 迷你 A2C。）

2. **项目二（SB3）**：任务二选一 `LunarLander-v3`（离散→DQN+PPO 都可）或 `HalfCheetah` 简化（PPO 为主）：
   - DQN（若离散）与 PPO 各 3 个 seed；
   - 输出：test 曲线（滑动平均）、最优/最差轨迹 GIF、超参消融（LR × entropy_coef 或 buffer_size）；
   - 报告 1 页：曲线 + 结论 + "什么任务选 value-based vs policy-based"。

## 阶段考 3 范围（W9–W13）

- semi-gradient 定义 + DQN 更新式 + 双网络机制（3 题）
- 死亡三联症 + 发散机制（2 题）
- λ-return 递归式 + trace 直觉（2 题）
- PG theorem 推导（给 3 个提示）+ REINFORCE + baseline 无偏证明思路（3 题）
- actor-critic 结构 + 连续动作高斯参数化（2 题）

## 常见坑

- **REINFORCE 的 loss 符号写反**（应为 `−Σ logπ·G` 的均值，即最大化 Σ logπ·G）→ 越训越差。
- baseline 用了"含动作的函数"（如 Q(s,a) 当 b 且 a 是采样的那个）→ 无偏性被破坏（b 必须与 a 独立）。
- G 没做 discount（0.99 漏乘）→ 早期 reward 权重虚高，曲线抖。
- PPO 的 `clip_range` 太大（>0.3）→ 策略每步跳太猛，训练崩（先 0.2）。
- 评估用"训练时的随机策略"不固定 seed → 曲线不可比；评估时 greedy/固定 seed。
- 手写 REINFORCE 与 SB3 PPO 对比时，把"算法差异"和"实现/超参差异"混为一谈 → 消融要控制变量。

## 自测

1. 为什么连续动作任务更适合 policy gradient？
2. 写出 PG theorem 的采样估计式（REINFORCE）。
3. baseline 为什么能降方差又不引入 bias？
4. actor 与 critic 各是什么？advantage 从哪来？
5. 高斯策略的 log 概率梯度里，σ 固定时 ∇log π 与 (a−μ) 的关系？

### 参考答案

1. 价值方法要 argmax 连续动作（难）且策略被硬投影成确定性；PG 直接参数化（随机）策略，连续动作可用高斯输出。
2. ∇J ≈ (1/T)Σ_t ∇log π_θ(A_t|S_t)·G_t（MC return 无 bias 高方差）。
3. b(s) 与 a 独立 → E[∇log π(a|s)·b(s)] = b(s)·Σ_a ∇π(a|s) = b(s)·∇(Σ_a π(a|s)) = 0（策略概率和为 1）；但 (G−b) 的方差随 b 接近 E[G|s] 而减小。
4. actor=π_θ（选动作），critic=V_φ（估 baseline/价值）；advantage ≈ r+γV(s')−V(s)（TD 估计的 Q−V）。
5. ∇_μ log N(a;μ,σ²) = (a−μ)/σ² → 梯度方向 = "把均值推向被采样的好动作"（乘以 advantage 后加权）。

## 笔记清单（对照书本补写）

- [ ] 抄写 PG theorem + log-derivative trick 推导（3–5 行）
- [ ] 写 baseline 无偏性的证明思路（Σ_a π = 1 那一步）
- [ ] 记录 REINFORCE 手写版曲线 + 与 PPO 的对比图
- [ ] 画 actor-critic 结构图（数据流：s→actor→a→r,s'→critic→adv→actor 更新）
- [ ] 整理"value-based vs policy-based 选型表"（动作空间/随机性/样本效率）
