# GAE（Generalized Advantage Estimation）总结笔记

整理内容：
- Advantage / TD error 定义
- GAE 两种等价公式（加权和 vs 递推）
- `targets = advantage + v_old` 的含义
- done mask 的正确用法
- 最小数值例子

## 1. Advantage 的基本定义



$$
A_t = Q(s_t,a_t) - V(s_t)
$$



直觉：在状态 $s_t$ 下，这次动作/轨迹带来的回报比基线 $V(s_t)$ 好多少/差多少。

## 2. 一步 TD 误差（TD error）



$$
\delta_t = r_{t+1} + \gamma\,(1-d_t)\,V(s_{t+1}) - V(s_t)
$$



- $d_t=1$ 表示终止，则 $(1-d_t)=0$，终止时不 bootstrap。
- 实现推荐用 `not_done = 1 - done`。

## 3. n-step Advantage（理解 GAE 的桥梁）

n-step return（带 bootstrap）：



$$
G_t^{(n)} = \sum_{l=0}^{n-1}\gamma^l r_{t+1+l} + \gamma^n (1-d_{t:t+n-1}) V(s_{t+n})
$$



n-step advantage：



$$
A_t^{(n)} = G_t^{(n)} - V(s_t)
$$



## 4. GAE 的两种等价公式

### 4.1 加权和形式（定义式）



$$
A_t^{\mathrm{GAE}(\gamma,\lambda)}=(1-\lambda)\sum_{n=1}^{\infty}\lambda^{n-1}A_t^{(n)}
$$



（有限轨迹里会截断到 episode 结束）

### 4.2 递推形式（实现最常用，从后往前）



$$
A_t = \delta_t + \gamma\lambda(1-d_t)A_{t+1}
$$



终止处 $(1-d_t)=0$，因此 $A_t=\delta_t$。

## 5. λ 的作用：Bias / Variance Trade-off

- $\lambda=0$：$A_t=\delta_t$（TD(0)，低方差/可能偏差大）
- $\lambda\to 1$（episode 有终止）：更接近 MC advantage：$A_t\approx G_t-V(s_t)$（低偏差/方差更大）

## 6. 为什么 critic target = advantage + V(s)

actor 用 $A_t$；critic 要回归的是回报目标（λ-return）：



$$
R_t^{\lambda} = A_t^{\mathrm{GAE}} + V_{\mathrm{old}}(s_t)
$$



对应实现：

```python
targets = stop_gradient(advantage + v_old)
```

## 7. 最小数值例子（手算）

设：$\gamma=0.9,\ \lambda=0.95$，三步轨迹 $t=0,1,2$，第 2 步后终止。

- $V(s_0)=1.0,\ V(s_1)=0.5,\ V(s_2)=0.2,\ V(s_3)=0$
- 奖励：$r_1=1.0,\ r_2=0.0,\ r_3=2.0$
- mask：$not\_done=[1,1,0]$

**TD error**：

- $\delta_0 = 1.0 + 0.9\cdot 0.5 - 1.0 = 0.45$
- $\delta_1 = 0.0 + 0.9\cdot 0.2 - 0.5 = -0.32$
- $\delta_2 = 2.0 - 0.2 = 1.8$（终止不 bootstrap）

**GAE 递推**（$\gamma\lambda=0.855$）：

- $A_2=1.8$
- $A_1=-0.32 + 0.855\cdot 1.8=1.219$
- $A_0=0.45 + 0.855\cdot 1.219\approx 1.492$

**critic targets**：$R_t=A_t+V(s_t)$：

- $R_0\approx 2.492,\ R_1\approx 1.719,\ R_2=2.0$

## 8. 实现要点速查

1) done mask：用 `not_done = 1 - done`。  
2) 递推：`A_t = delta_t + gamma*lam*not_done_t*A_{t+1}`。  
3) actor 用 advantage；critic 用 `targets = advantage + v_old`。  
4) advantage normalization 常用于稳定 actor 更新（不改变 critic target 的定义）。
