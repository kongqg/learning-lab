# MDP（Markov Decision Process）复习笔记

> 主题：用 MDP 形式化“序列决策 / 控制”问题；RL 是在未知或部分未知 MDP 模型下，通过采样学习策略的算法体系。

---

## 1. 为什么要有 MDP？

因为RL要解决的是短视的问题，如果遇到带有序列决策的问题，是适合用RL去解决的，MDP是定义为数学问题

MDP 的核心作用是：**把“多步决策问题”抽象成可计算的数学对象**，从而能定义“最优策略/最优回报”，并推导出 Bellman 递推等关键方程。

典型场景特征：
- 决策是**连续多步**的，而不是一次性选择
- 当前动作会影响未来状态（影响未来可获得奖励）
- 环境可能有随机性（不确定性）
- 目标是**最大化长期累计回报**（而非单步奖励）

> 一句话：**MDP 定义问题；RL 在 MDP 上求解/学习。**

---

## 2. MDP 的五元组定义

一个（折扣）MDP 通常表示为五元组：

$$
\mathcal{M} = (S, A, P, R, \gamma)
$$


- \(S\)：状态空间（state space）
- \(A\)：动作空间（action space）
- \(P\)：**状态转移概率分布 / 转移核**（transition kernel）
  $$
  P(s' \mid s,a)=\Pr(S_{t+1}=s' \mid S_t=s, A_t=a)
  $$
  
- \(R\)：奖励函数（reward）
  - 常见写法：\(R(s,a)\) 或随机奖励分布 \(
    $$
    P(r \mid s,a)
    $$
    \)，也可写成期望奖励
    $$
    r(s,a)=\mathbb{E}[R_{t+1}\mid s,a]
    $$
    
- 折扣因子（discount factor）

---

## 3. 马尔可夫性（Markov Property）

马尔可夫性强调的是：**给定当前状态与动作，下一步状态（以及奖励）的分布与更早历史无关**。
$$
P(S_{t+1}\mid S_t, A_t) = P(S_{t+1}\mid S_0,A_0,\dots,S_t,A_t)
$$
若奖励也随机，则：
$$
P(R_{t+1}\mid S_t, A_t) = P(R_{t+1}\mid S_0,A_0,\dots,S_t,A_t)
$$

---

## 4. 策略（Policy）

策略 \(\pi\) 描述“在状态下如何选动作”。

### 4.1 随机策略（stochastic policy）
$$
\pi(a\mid s)=\Pr(A_t=a \mid S_t=s)
$$



### 4.2 确定性策略（deterministic policy）
$$
\mu(s)=a
$$

---

## 5. 轨迹（Trajectory）

在策略 \(\pi\) 与环境动力学 \(P\) 下，系统会生成一条随机轨迹：
$$
\tau=(S_0,A_0,R_1,S_1,A_1,R_2,\dots,S_T)
$$
（奖励的下标也常写成 \(R_t\)，只要在整套推导中保持一致即可。）

---

## 6. 回报（Return）

从时刻 \(t\) 起的折扣累计回报定义为：
$$
G_t=\sum_{k=0}^{\infty}\gamma^k R_{t+k+1}
$$
有限时域（episode 长度为 \(T\)）：
$$
G_t=\sum_{k=0}^{T-t-1}\gamma^k R_{t+k+1}
$$

---

## 7. 价值函数（Value Functions）

### 7.1 状态价值函数
$$
V^\pi(s)=\mathbb{E}_\pi\left[\,G_t\mid S_t=s\,\right]
$$



### 7.2 动作价值函数
$$
Q^\pi(s,a)=\mathbb{E}_\pi\left[\,G_t\mid S_t=s, A_t=a\,\right]
$$

两者关系：
$$
V^\pi(s)=\sum_a \pi(a\mid s)\,Q^\pi(s,a)
$$


---

## 8. RL 与 MDP 的关系（定位）

- **MDP**：定义“序列决策问题”本身（状态、动作、转移、奖励、折扣）
- **RL**：当 \(P\)（以及可能的 \(R\)）未知或难以精确建模时，通过与环境交互采样、利用 \(G,V,Q\) 等对象去学习策略 \(\pi\)

---
