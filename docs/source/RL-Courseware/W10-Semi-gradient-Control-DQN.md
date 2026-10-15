# W10 · 半梯度控制：DQN（课件）

> 对应阅读: Ch10 On-policy Control with Approximation (PDF p.265–278)，重点 10.2 semi-gradient n-step Sarsa、10.3–10.4 DQN 要点
> 里程碑: CartPole/Breakout 的 DQN（手写简化版 + SB3 对比）
> 配套笔记: 建议 `Chapter-10-DQN-and-Semi-gradient.md`（待写）

## 学习目标

1. 掌握 semi-gradient Q-learning 的更新（神经网络版 Q-learning）；
2. 说清 DQN 两大稳定化手段：experience replay + target network；
3. 用 SB3 跑通 DQN 基线，并与手写简化版对比；
4. 理解 average reward（continuing 任务的现代设定）。

## 核心概念

### 1. Semi-gradient Q-learning（10.2，DQN 的算法核心）

```
δ = r + γ max_a' Q(s',a'; w⁻) - Q(s,a; w)
w ← w + α ∇_w Q(s,a; w) * δ          (只对 Q(s,a) 求导, target detach)
```

= Q-learning 的 bootstrap + 神经网络参数化 + semi-gradient 更新。

### 2. 为什么不直接跑（发散根源，Ch11 铺垫）

**死亡三联症** = 函数近似 + bootstrapping + off-policy，三者齐上会发散（W11 详述）。
DQN 的"工程解药"（不是理论证明，是实用稳定化）：

| 手段 | 作用 |
|---|---|
| **Experience Replay** | 打散样本时间相关性 → 降低方差、复用样本 |
| **Target Network** w⁻（慢更新） | 让 target 近似固定 → 避免"追着移动靶打" |

### 3. n-step Sarsa 与 DQN 的关系（10.2）

n-step Sarsa 的 semi-gradient 版 = "n 步 Q-learning 家族"的一支；DQN 取 n=1 + off-policy（max）。→ 理解"DQN = off-policy + bootstrap + 近似"这三件齐活，就理解了它为什么需要那两剂解药。

### 4. Average Reward（10.3–10.4，2nd ed. 新设定）

continuing 任务 γ=1 时 return 发散 → 改用**平均 reward** `J(π) = lim (1/T) Σ r_t` 作为目标；
- 价值函数换成"差分值"（relative value），基线（bias）自由；
- 实践中：γ<1 的 DQN/PPO 仍是主流，但要知道理论上的现代设定（考试爱考）。

## 动手步骤

1. **手写简化 DQN**（CartPole-v1，~60 行）：

```python
import torch, torch.nn as nn, random
import numpy as np, gymnasium as gym

class QNet(nn.Module):
    def __init__(self, n_in, n_act):
        super().__init__()
        self.net = nn.Sequential(nn.Linear(n_in, 64), nn.ReLU(), nn.Linear(64, n_act))
    def forward(self, x): return self.net(x)

env  = gym.make("CartPole-v1")
qnet = QNet(env.observation_space.shape[0], env.action_space.n)
qtgt = QNet(env.observation_space.shape[0], env.action_space.n)
qtgt.load_state_dict(qnet.state_dict()); qtgt.eval()

replay = []
opt = torch.optim.Adam(qnet.parameters(), 1e-3)
for ep in range(1000):
    s, _ = env.reset(seed=ep); done = False; ep_r = 0
    while not done:
        qs = qnet(torch.tensor(s, dtype=torch.float32))
        a = int(np.argmax(qs.detach().numpy())) if random.random() > 0.05 else random.randrange(2)
        sp, r, terminated, truncated, _ = env.step(a); done = terminated or truncated
        replay.append((s, a, r, sp, int(done)))
        if len(replay) > 50000: replay.pop(0)

        if len(replay) > 128:
            b = random.sample(replay, 128)
            s_b = torch.tensor(np.array([x[0] for x in b]), dtype=torch.float32)
            a_b = torch.tensor(np.array([x[1] for x in b]), dtype=torch.long)
            r_b = torch.tensor(np.array([x[2] for x in b]), dtype=torch.float32)
            sp_b = torch.tensor(np.array([x[3] for x in b]), dtype=torch.float32)
            d_b = torch.tensor(np.array([x[4] for x in b]), dtype=torch.float32)
            with torch.no_grad():
                nxt = qtgt(sp_b).max(1)[0]
                tgt = r_b + 0.99 * nxt * (1 - d_b)
            loss = ((qnet(s_b).gather(1, a_b.view(-1,1)).squeeze(1) - tgt) ** 2).mean()
            opt.zero_grad(); loss.backward(); opt.step()
            ep_r += r
    if ep % 50 == 0:
        qtgt.load_state_dict(qnet.state_dict())
    print(ep, ep_r)
```

2. **SB3 基线**：

```python
from stable_baselines3 import DQN
from stable_baselines3.common.evaluation import evaluate_policy
model = DQN("MlpPolicy", "CartPole-v1", learning_rate=1e-4, buffer_size=50_000, verbose=1)
model.learn(250_000)
evaluate_policy(model, gym.make("CartPole-v1"))
```

3. 对比：手写 vs SB3 的 test 曲线（同 seed 集 3 个）；消融：去掉 replay / 去掉 target net 各跑一次 → 记录崩溃/漂移。

## 常见坑

- **target 忘了 detach/no_grad** → 反传穿过 target，loss 数值飘、不收敛（最常见 bug）。
- replay buffer 太小（<512）→ 样本相关性没打散，曲线抖动。
- target net 每步都同步（而不是每 N 步/软更新）→ 失去"慢"的意义，退化为单网络（易震荡）。
- CartPole 的 reward 是步数（每步 +1）→ DQN 会"无限拖延"刷步数，500 步上限前都算"赢"；评估要看 500 步达成率。
- 手写版 ε 衰减没做（一直 5% 随机）→ 后期策略"抖"；加 ε: 0.9→0.05 线性衰减。
- SB3 的 `MlpPolicy` 默认 2×64，改网络结构要用 `policy_kwargs`，别乱改 API。

## 自测

1. DQN 的"死亡三联症"占了哪几件？两剂"解药"分别对付什么？
2. experience replay 打散了什么？
3. target network 为什么要"慢更新"？
4. semi-gradient Q-learning 的更新式（写出 δ 与梯度方向）？
5. average reward 设定解决了 γ=1 continuing 任务的什么问题？

### 参考答案

1. 三件全占（近似 + bootstrap + off-policy）；replay 降方差/去相关性，target net 固定化 target 降"移动靶"方差。
2. 相邻样本的强相关性（同一起始条件的连锁反应）→ 采样后近似 i.i.d.。
3. 让 TD target 在更新期间近似常数，避免"目标跟着参数跑"的正反馈发散；软更新（polyak）是平滑版。
4. δ = r + γ max Q(s',w⁻) − Q(s,a;w)；w ← w + αδ·∇_w Q(s,a;w)（只对 Q(s,a) 项求导）。
5. γ=1 时 return 无穷（发散）；average reward 用"长期平均"代替折现和，使目标良定义（价值函数需重定义为 relative value）。

## 笔记清单（对照书本补写）

- [ ] 抄写 DQN 更新式 + replay/target 的机制图
- [ ] 记录手写 vs SB3 曲线 + 消融实验（去 replay / 去 target）结果
- [ ] 用自己的话写"DQN = Q-learning + 神经网络 + 两剂稳定药"
- [ ] 整理 ε 衰减方案（线性/指数）+ 你选的参数
- [ ] 写一段 average reward vs discounted return 的对比（各 2 句）
