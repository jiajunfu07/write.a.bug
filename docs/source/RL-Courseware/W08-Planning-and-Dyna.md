# W08 · 规划：Dyna 与 MCTS + 阶段考 2（课件）

> 对应阅读: Ch8 Planning and Learning with Tabular Methods (PDF p.181–216)，重点 8.2 Dyna、8.4 模型错了、8.7–8.11（RTDP/rollout/MCTS）
> 里程碑: Dyna vs Q 对比 + 玩具 MCTS（井字棋）+ 阶段考 2
> 配套笔记: 建议 `Chapter-8-Planning-and-Dyna.md`（待写）

## 学习目标

1. 掌握 Dyna 的"学模型 + 想象训练"机制，说清它为什么样本效率高；
2. 理解"模型错了"时的偏差来源与 mitigations；
3. 理解规划（decision-time search）与学习的分工；
4. 实现玩具 MCTS 并击败随机井字棋玩家。

## 核心概念

### 1. 模型（8.1）

模型 = 对环境的近似：奖励模型 R(s,a) + 转移模型 P(s'|s,a)。
**模型可以从数据学**（计数/最大似然）——这就是"model-based RL"的入口。

### 2. Dyna（8.2，"integrated architecture"）

```
真实交互 ──▶ ① 学模型 (更新 R, P)
            ② 真实 TD 更新 (用真实经历)
            ③ 模拟 TD 更新 (用模型"想象" k 步)   ← 样本效率来源
```

- 一条真实轨迹 → 3 份学习信号（真实 + 想象）；
- 代价：想象基于错误模型会引入 bias（8.4）；
- 实践要点：k（每步想象多少）太小没收益、太大 bias 大——扫一遍。

### 3. 当模型错了（8.4）

- 模型误差 → 想象更新系统性偏移 → 最终策略不是最优（bias，不随样本数消失）；
- 缓解：k 调小、模型置信度加权（8.5 Prioritized Sweeping 的"按重要度更新"思想）。

### 4. Prioritized Sweeping（8.5，概念层）

TD 更新只动了一个状态；但它影响的上游状态（predecessors）也该更新 → 按"影响度"排序批量传播。直觉：**一个格子变了，把"通往它的格子"也重新算一遍**。

### 5. 规划 vs 学习（8.7–8.11）

- **学习**：离线把 Q 学好（训练阶段）；
- **规划**：决策时现场搜索（rollout / MCTS / heuristic search）；
- **MCTS（8.11）** 四步循环：
  1. **Select**：按 UCB（树内）从根走到叶；
  2. **Expand**：加一个未访问子节点；
  3. **Simulate (rollout)**：从该节点随机走到终局；
  4. **Backpropagate**：把结果沿路径回传更新 Q/N。
- 妙处：选择用 W2 的 UCB（探索-利用），评估用 MC（rollout）——前面所有工具的合体。

## 动手步骤

1. **Dyna-Q vs Q-learning**（同一 gridworld）：
   - Dyna：每步真实交互后，用模型"想象" k=5 步做 Q 更新；
   - 对比：达到 0.9×最优 return 的**真实交互步数**（Dyna 应显著更少）；
   - 画 Q 的 bias 对比（Dyna 最终 Q vs 真 q*）。
2. **玩具 MCTS（井字棋）**：

```python
import random, math
children = {}      # node -> [child_nodes]
Q, N = {}, {}      # node -> {action: (q, n)}

def select(node):
    while is_terminal(node) is False and children.get(node):
        node = max(children[node], key=lambda c: ucb(c))   # UCB 选择
    return node

def mcts(root, iters=1000):
    for _ in range(iters):
        node = select(root)
        if not is_terminal(node):
            node = expand(node)
        result = rollout(node)              # 随机走到底, +1/0/-1
        backpropagate(node, result)
    return best_action(root)                # 选 N 最大的孩子
```

   验收：MCTS（iters=1000）对随机玩家胜率 > 90%；对"最优贪心"不输。

## 阶段考 2 范围（W3–W8）

- Bellman 方程 / V*/q* / 最优策略存在性（3 题）
- DP 两迭代 + GPI（2 题）
- MC 的 ES/ε-soft/IS（2 题）
- TD/Sarsa/Q/double Q 更新式 + 悬崖问题（3 题）
- n-step 谱系 + Tree Backup（2 题）
- Dyna 三通道 + 模型偏差 + MCTS 四步（3 题）

## 常见坑

- Dyna 的"想象更新"和"真实更新"用了不同的 Q 表 → 学飞了（必须同一张表）。
- 模型计数没平滑（冷启动 N=0 除零）→ 加 prior（最大似然 + 均分初始）。
- MCTS 的 rollout 用"贪心策略"会严重偏置（早期）→ 玩具版先随机 rollout（书 8.11 的 baseline）。
- UCB 的 c 值：井字棋 c=1.41 附近即可，太大探索过度、太小利用过度（扫 c 看看）。
- 把"规划步数"和"训练步数"混着算样本效率 → 分开统计（真实交互 vs 想象/搜索）。

## 自测

1. Dyna 的三条学习信号分别是什么？
2. 为什么 Dyna 样本效率高、但可能有 bias？
3. Prioritized Sweeping 解决 TD 的什么"浪费"？
4. MCTS 四步 + 每步用到的前面学过的工具。
5. "学习"与"规划"的分工边界在哪里（什么任务用哪个）？

### 参考答案

1. 真实 TD（实际经历）+ 模型学习（R,P）+ 模拟 TD（想象 k 步）。
2. 每条真实轨迹派生多条"想象"轨迹，样本效率 ×(1+k)；但想象来自近似模型，误差会累积成 bias。
3. TD 只更新当前状态，影响应传播到上游；sweeping 按重要度批量更新 predecessors，减少"反复路过才更新"的浪费。
4. Select(UCB)→Expand(新节点)→Simulate(MC rollout)→Backprop(更新 Q,N)；分别用 W2 的 UCB 与 W5 的 MC。
5. 状态空间小/可枚举/决策时算得起 → 规划（棋类）；状态空间大/在线实时 → 学习（机器人）；两者可混合（Dyna/MCTS+学习）。

## 笔记清单（对照书本补写）

- [ ] 画 Dyna 架构图（真实/模型/想象三通道）
- [ ] 记录 Dyna vs Q 的样本效率对比表（k 扫描）
- [ ] 抄写 MCTS 四步 + 每步伪代码
- [ ] 记录 MCTS 井字棋胜率（vs 随机 / vs 贪心）+ 参数
- [ ] 用自己的话写"model-based vs model-free"决策树（什么场景选什么）
