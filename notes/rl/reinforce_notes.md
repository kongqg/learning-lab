# REINFORCE（Policy Gradient）笔记 + 伪代码（含 Reward-to-Go 版本）

> 关键词：**Likelihood-Ratio Trick**、**Monte Carlo Policy Gradient**、**Baseline**、**Reward-to-Go**、**High Variance**、**On-policy**、**Entropy（探索）**  
> 目标：把 REINFORCE 的“能跑/能解释/知道坑”一次性打通。

---

---

## 1. REINFORCE 核心目标与梯度

### 1.1 最基本形式（无 baseline）
$$
\nabla_\theta J(\theta)=\mathbb{E}\Big[\sum_{t} \nabla_\theta \log \pi_\theta(a_t|s_t)\, G_t\Big]
$$
其中
$$
G_t=\sum_{k=0}^{T-t-1}\gamma^k r_{t+1+k}
$$

**直觉**：回报高 ⇒ 增加该动作概率；回报低 ⇒ 降低该动作概率。

---

---

## 2. 为什么 REINFORCE 对连续动作更自然？

连续动作下，价值方法（如 Q-learning）常需要：

$$
\max_{a} Q(s,a)
$$

这是一个连续优化问题（高维时困难且不稳定）。

REINFORCE 直接参数化策略分布，例如：

$$
\pi_\theta(a|s)=\mathcal N(\mu_\theta(s),\sigma_\theta(s))
$$

从分布采样动作并更新分布参数，无需解 $\arg\max$。

---

---

## 3. Baseline：降方差但不改变期望梯度

### 3.1 加 baseline 的形式

$$
\nabla_\theta J(\theta)=\mathbb{E}\Big[\sum_{t} \nabla_\theta \log \pi_\theta(a_t|s_t)\, (G_t-b(s_t))\Big]
$$

### 3.2 为什么不改变期望梯度？
关键性质：

$$
\mathbb{E}_{a\sim \pi_\theta(\cdot|s)}[\nabla_\theta \log \pi_\theta(a|s)] = 0
$$

因此若 $b(s)$ 与动作 $a$ 无关：

$$
\mathbb{E}[\nabla_\theta \log \pi_\theta(a|s)\, b(s)] = b(s)\cdot 0 = 0
$$

所以 baseline 只降方差，不引入偏差。

> **关键条件（易忽略）**：baseline **不能依赖动作 $a_t$**（可以依赖状态/时间/历史）。

---

---

## 4. Reward-to-Go：一个“经常被忽略但很关键”的方差降低技巧

### 4.1 为什么需要 Reward-to-Go？
原始 REINFORCE 有时会给整条轨迹所有时间步用同一个总回报，这会把很多“与当前动作无关的未来噪声”乘进梯度 ⇒ 方差更大。

### 4.2 Reward-to-Go 的定义

$$
G^{\text{rtg}}_t=\sum_{k=0}^{T-t-1}\gamma^k r_{t+1+k}
$$

即从时刻 $t$ 开始的“后缀回报”，对更早的动作更公平，也显著降方差（期望不变）。

---

---

## 5. On-policy：为什么 REINFORCE 不能直接反复用旧数据？

REINFORCE 的期望是对当前策略分布取的：

$$
\mathbb{E}_{\tau\sim \pi_\theta}[\cdot]
$$

如果用旧策略 $\pi_{\text{old}}$ 的数据直接估计，会得到偏的梯度（除非用重要性采样修正）。

---

---

## 6. 容易忽略但对理解很重要的点（Checklist）

1) **学的是概率分布参数**（连续动作是 $\mu,\sigma$，不是直接回归动作值）  
2) **不 bootstrap** ⇒ 近似无偏，但方差大（MC 特性）  
3) **Credit assignment 难** ⇒ Reward-to-Go、baseline、advantage 是必须的  
4) **探索依赖策略熵**：策略过早变尖锐会探索塌陷，常加 entropy bonus（工程必备）  
5) **baseline 不许依赖动作**，否则引入偏差

---

# 7. 伪代码

---

## 7.1 REINFORCE（最基本 / 轨迹总回报版本）
> 注意：这是“教学版”，方差更大；更推荐用 Reward-to-Go。

```text
Initialize policy parameters θ

for iteration = 1..:
    Collect N episodes using current policy π_θ
    for each episode τ:
        Compute total return G = sum_{t=0}^{T-1} γ^t r_{t+1}

        for t = 0..T-1:
            # same G used for all timesteps (high variance)
            θ ← θ + α * ∇_θ log π_θ(a_t | s_t) * G
```

---

## 7.2 REINFORCE + Reward-to-Go（推荐）
```text
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

---

## 7.3 REINFORCE + Reward-to-Go + Baseline（最常用形态）
```text
Initialize policy parameters θ
Initialize baseline function b(·)  (e.g., state-value V_φ)

for iteration = 1..:
    Collect N episodes using current policy π_θ
    for each episode τ:
        # backward compute reward-to-go
        G = 0
        for t = T-1 down to 0:
            G = r_{t+1} + γ * G
            advantage = G - b(s_t)      # baseline must NOT depend on a_t
            θ ← θ + α * ∇_θ log π_θ(a_t | s_t) * advantage

        # (optional) update baseline parameters to fit returns:
        # φ ← argmin_φ Σ_t (b(s_t) - G_t)^2
```

---

---

## 8. 记忆卡片（只背这 5 行）

- REINFORCE：

$$\nabla \log \pi \times$$

return  
- 连续动作友好：不需要 $\arg\max_a Q(s,a)$，直接学分布  
- baseline：$G_t-b(s_t)$，**降方差不改期望**（b 不依赖 a）  
- Reward-to-Go：用后缀回报 $G_t$ 代替全轨迹总回报，方差更小  
- on-policy：数据必须来自当前策略（否则梯度偏，需重要性采样）

---

*生成时间：2026-01-13 03:30:25*