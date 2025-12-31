# Bellman 方程与最优性（Bellman Expectation / Optimality）复习笔记

> 本节目标：搞清楚 **V vs Q**、**策略评估 vs 最优控制**，以及 Bellman 递推为什么会出现“对动作求和/取 max”。

---

## 1. V 与 Q 的核心区别（Definition）

- **状态价值函数**
  $$
  V^\pi(s)=\mathbb{E}_\pi[G_t\mid S_t=s]
  $$
  

- **动作价值函数**
  $$
  Q^\pi(s,a)=\mathbb{E}_\pi[G_t\mid S_t=s, A_t=a]
  $$
  

- **连接关系（非常重要）**
  $$
  V^\pi(s)=\sum_a \pi(a\mid s)\,Q^\pi(s,a)
  $$
  

直觉：
- \(V^\pi(s)\)：当前动作未指定 → 必须按策略对动作求期望

- \(Q^\pi(s,a)\)：当前动作已指定为 \(a\) → 外层不再对当前动作求期望，但是里层还需要求和，因为策略是随机的，只有当argmax的时候，每一步才是固定的
  $$
  Q^\pi(s,a)=\sum_{s'}P(s'\mid s,a)\Big(r(s,a,s')+\gamma \sum_{a'}\pi(a'\mid s')Q^\pi(s',a')\Big)
  $$
  

---

## 2. Bellman 期望方程（固定策略 \(\pi\)：策略评估 Prediction）

### 2.1 \(V^\pi\) 的 Bellman 期望方程
$$
V^\pi(s)=\sum_a \pi(a\mid s)\sum_{s'}P(s'\mid s,a)\Big(r(s,a,s')+\gamma V^\pi(s')\Big)
$$



### 2.2 \(Q^\pi\) 的 Bellman 期望方程
$$
Q^\pi(s,a)=\sum_{s'}P(s'\mid s,a)\Big(r(s,a,s')+\gamma \sum_{a'}\pi(a'\mid s')Q^\pi(s',a')\Big)
$$



> 为什么 \(Q^\pi\) 里还有
> $$
> \sum_{a'}\pi(a'|s')
> $$
> ？
> - 因为 **只固定了当前步动作 \(A_t=a\)**；
> - 下一步动作仍按策略抽样
>   $$
>   A_{t+1}\sim\pi(\cdot|S_{t+1})
>   $$
>   ，所以在时刻 \(t\) 写递推时，需要对 \(a'\) 取期望。
> - 若 \(\pi\) 是确定性策略 \(\mu\)，则该求和变为
>   $$
>   Q^\pi(s',\mu(s'))
>   $$
>   （求和“消失”）。

---

## 3. Bellman 最优性方程（\(^*\)：最优控制 Control）

### 3.1 最优状态价值 \(V^*\)
$$
V^*(s)=\max_a \sum_{s'}P(s'\mid s,a)\Big(r(s,a,s')+\gamma V^*(s')\Big)
$$

### 3.2 最优动作价值 \(Q^*\)
$$
Q^*(s,a)=\sum_{s'}P(s'\mid s,a)\Big(r(s,a,s')+\gamma \max_{a'}Q^*(s',a')\Big)
$$

直觉：
- 从 \(V^\pi\) 到 \(V^*\)：**“按策略平均”**（\(\sum_a\pi(a|s)\)）→ **“选最优动作”**（\(\max_a\)）
- 但要注意：环境如果随机，**下一状态仍需要期望/求和**；不是“只有一条确定路线”。

---

## 4. 从 \(Q^*\) 构造最优策略（Greedy / Argmax）

### 4.1 确定性最优策略（最常用）
$$
\pi^*(s)=\arg\max_{a} Q^*(s,a)
$$



### 4.2 并列最优动作（可选）
令
$$
\mathcal{A}^*(s)=\{a:\; Q^*(s,a)=\max_{a'}Q^*(s,a')\}
$$

则任何只在
$$
\mathcal{A}^*(s)
$$

 上分配概率的策略都是最优的（例如均匀分配）。

