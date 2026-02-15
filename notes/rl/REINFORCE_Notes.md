# REINFORCE (Policy Gradient) 笔记

**关键词：** `Likelihood-Ratio Trick`、`Monte Carlo Policy Gradient`、`Baseline`、`Reward-to-Go`、`High Variance`、`On-policy`、`Entropy`

---

## 1. REINFORCE 核心目标与梯度

### 1.1 最基本形式（无 baseline）
$$
abla_	heta J(	heta)=\mathbb{E}\Big[\sum_{t} 
abla_	heta \log \pi_	heta(a_t|s_t) \cdot G_t\Big]$$
其中 $$G_t=\sum_{k=0}^{T-t-1}\gamma^k r_{t+1+k}$$

> **直觉：** 回报高 $\Rightarrow$ 增加该动作概率；回报低 $\Rightarrow$ 降低该动作概率。

---

## 2. 为什么 REINFORCE 对连续动作更自然？

在连续动作下，价值方法（如 Q-learning）常需要：
$$\max_{a} Q(s,a)$$
这是一个连续优化问题（高维时困难且不稳定）。

REINFORCE 直接参数化策略分布，例如：
$$\pi_	heta(a|s)=\mathcal{N}(\mu_	heta(s),\sigma_	heta(s))$$
从分布采样动作并更新分布参数，无需解 $rg\max$。

---

## 3. Baseline：降方差但不改变期望梯度

### 3.1 加 baseline 的形式
$$
abla_	heta J(	heta)=\mathbb{E}\Big[\sum_{t} 
abla_	heta \log \pi_	heta(a_t|s_t) \cdot (G_t-b(s_t))\Big]$$

### 3.2 为什么不改变期望梯度？
关键性质：
$$\mathbb{E}_{a\sim \pi_	heta(\cdot|s)}[
abla_	heta \log \pi_	heta(a|s)] = 0$$
因此若 $b(s)$ 与动作 $a$ 无关：
$$\mathbb{E}[
abla_	heta \log \pi_	heta(a|s) \cdot b(s)] = b(s) \cdot 0 = 0$$
所以 baseline 只降方差，不引入偏差。

> **注意：** baseline 不能依赖动作 $a_t$（可以依赖状态/时间/历史）。

---

## 4. Reward-to-Go：降方差的关键技巧

### 4.1 为什么需要 Reward-to-Go？
原始 REINFORCE 有时会给整条轨迹所有时间步用同一个总回报，这会把很多“与当前动作无关的未来噪声”乘进梯度 $\Rightarrow$ 方差更大。

### 4.2 定义
$$G^{	ext{rtg}}_t=\sum_{k=0}^{T-t-1}\gamma^k r_{t+1+k}$$
即从时刻 $t$ 开始的“后缀回报”，对更早的动作更公平，也显著降方差（期望不变）。

---

## 5. On-policy：数据的局限性
REINFORCE 的期望是对当前策略分布取的：
$$\mathbb{E}_{	au\sim \pi_	heta}[\cdot]$$
如果用旧策略 $\pi_{	ext{old}}$ 的数据直接估计，会得到偏的梯度（除非用重要性采样修正）。

---

## 6. 容易忽略的 Checkpoints
* **学的是分布参数：** 连续动作是 $(\mu,\sigma)$，不是直接回归动作值。
* **不 bootstrap：** 近似无偏，但方差大（MC 特性）。
* **Credit assignment 难：** Reward-to-Go、baseline 是必须的。
* **探索依赖策略熵：** 策略过早变尖锐会探索塌陷，常加 **Entropy Bonus**。
* **baseline 独立性：** 绝不许依赖动作。

---

## 7. 伪代码

### 7.1 REINFORCE + Reward-to-Go（推荐）
```python
Initialize policy parameters θ

for iteration = 1..:
    Collect N episodes using current policy π_θ
    for each episode τ:
        # compute reward-to-go by backward scan
        G = 0
        for t = T-1 down to 0:
            G = r_{t+1} + γ * G
            # this G is G_t^{rtg}
            θ ← θ + α * ∇_θ log π_θ(a_t | s_t) * G
```

### 7.2 REINFORCE + Reward-to-Go + Baseline（标准）
```python
Initialize policy parameters θ
Initialize baseline function b(·)

for iteration = 1..:
    Collect N episodes using current policy π_θ
    for each episode τ:
        G = 0
        for t = T-1 down to 0:
            G = r_{t+1} + γ * G
            advantage = G - b(s_t)
            θ ← θ + α * ∇_θ log π_θ(a_t | s_t) * advantage
```

---

## 8. 记忆卡片（5 行精华）
1.  **REINFORCE：** $
abla \log \pi 	imes$ return
2.  **连续动作友好：** 不需要 $rg\max_a Q(s,a)$，直接学分布
3.  **Baseline：** $G_t-b(s_t)$，降方差不改期望
4.  **Reward-to-Go：** 后缀回报去噪
5.  **On-policy：** 数据必须来自当前策略
