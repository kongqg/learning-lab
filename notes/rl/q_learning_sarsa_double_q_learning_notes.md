# Q-learning / SARSA / Double Q-learning 笔记 + 伪代码（速查版）

> 关键词：**Off-policy vs On-policy**、**Bootstrap**、**Max Overestimation Bias**、**GLIE**、**Behavior policy / Target policy**  
> 目标：把三个算法的“更新 target 差异”和“偏差来源”一次性打通。

---

## 1. Tabular Q-learning：学 $Q^\*$ 的经典 off-policy 控制

### 1.1 更新公式
对转移 $(s_t,a_t,r_{t+1},s_{t+1})$：

$$
Q(s_t,a_t)\leftarrow Q(s_t,a_t)+\alpha\Big(r_{t+1}+\gamma \max_{a'}Q(s_{t+1},a')-Q(s_t,a_t)\Big)
$$

目标（target）：

$$
y = r_{t+1}+\gamma \max_{a'}Q(s_{t+1},a')
$$

### 1.2 为什么是 off-policy？
- **采样动作**来自行为策略 $\mu$（常用 $\epsilon$-greedy 或随机）
- **更新目标**对应贪心策略 $\pi$（用 $\max_{a'}$ 假设下一步选最优）
- 行为策略 $\mu$ 与目标策略 $\pi$ 可以不同 ⇒ **off-policy**

---

## 2. SARSA：on-policy 控制（学 $Q^\pi$）

### 2.1 更新公式

$$
Q(s_t,a_t)\leftarrow Q(s_t,a_t)+\alpha\Big(r_{t+1}+\gamma Q(s_{t+1},a_{t+1})-Q(s_t,a_t)\Big)
$$

其中 $a_{t+1}$ 是 **行为策略实际执行的动作**（如 $\epsilon$-greedy 采样得到）。

### 2.2 为什么是 on-policy？
target 用的是 **实际执行动作** $a_{t+1}$，因此更新目标对应当前行为策略 $\pi$ 本身 ⇒ **on-policy**

---

## 3. Q-learning vs SARSA：最重要的区别（背诵版）

- Q-learning 的 target：$\;r+\gamma\max_{a'}Q(s',a')$
- SARSA 的 target：$\;r+\gamma Q(s',a_{t+1})$

**直觉**：  
- Q-learning 更“激进/乐观”（按最优动作评估未来）  
- SARSA 更“保守”（按实际会做的动作评估未来）

---

## 4. “Bias” 到底来自哪里？（易混点）

### 4.1 Bootstrap bias（两者都有）
两者都用估计的 $Q$ 构造 target（自举），不是用真实回报 $G_t$。  
因此即使同一批数据，也会有偏差来源于 bootstrap。

### 4.2 Max overestimation bias（Q-learning 更典型）
Q-learning 用 $\max$ 在有噪声的估计中挑最大，容易系统性高估：

$$
\mathbb{E}[\max_a \hat Q] \ge \max_a \mathbb{E}[\hat Q]
$$

这会导致 **过估计偏差（overestimation）**，也是 Double Q / Double DQN 的动机。

---

## 5. Double Q-learning：缓解 overestimation bias

### 5.1 核心思想（一句话）
> 用一套估计来 **选动作（argmax）**，另一套估计来 **评估该动作的值**，把“选择”和“评价”解耦，降低 max 造成的高估。

### 5.2 典型 target（交替更新）
维护两张表（或两套估计）$Q^A, Q^B$。

当更新 $Q^A$ 时：

$$
a^\* = \arg\max_{a'} Q^A(s',a')
$$

$$
y = r + \gamma Q^B(s', a^\*)
$$

（更新 $Q^B$ 时对称交换 A/B）

---

## 6. 行为策略（Behavior Policy）到底做什么？

常用 $\epsilon$-greedy：
- 以 $1-\epsilon$ 的概率选：$a=\arg\max_a Q(s,a)$
- 以 $\epsilon$ 的概率随机选动作（探索）

**注意**：行为策略通常需要用到当前 $Q$（查表/前向网络）来做贪心选择。

---

## 7. GLIE（Greedy in the Limit with Infinite Exploration）

一句话：**前期探索必须足够全，后期策略必须趋于贪心。**

两条硬条件：
1) **Infinite exploration**：每个 $(s,a)$ 都会被访问无限次  
2) **Greedy in the limit**：$\epsilon_t \to 0$（最终变贪心）

结论：
- 若 $\epsilon>0$ 恒定：SARSA 收敛到带探索的 $Q^{\pi_\epsilon}$（不是纯 $Q^\*$）
- 若满足 GLIE：表格条件下可收敛到最优（经典结果）

---

# 8. 伪代码（最常用版本）

## 8.1 Q-learning（tabular）伪代码
```text
Initialize Q(s,a) arbitrarily (e.g., 0 for all s,a)
for episode = 1..:
    s = reset()
    repeat:
        # behavior policy (e.g., epsilon-greedy)
        with prob epsilon: a = random_action()
        else:             a = argmax_a Q(s,a)

        take action a, observe r, s'
        # Q-learning target uses max over next-state actions
        target = r + gamma * max_{a'} Q(s', a')
        Q(s,a) = Q(s,a) + alpha * (target - Q(s,a))

        s = s'
    until terminal
```

## 8.2 SARSA（tabular）伪代码
```text
Initialize Q(s,a) arbitrarily
for episode = 1..:
    s = reset()
    # choose first action using behavior policy
    a = epsilon_greedy(Q, s)

    repeat:
        take action a, observe r, s'
        # choose next action using SAME behavior policy
        a_next = epsilon_greedy(Q, s')

        # SARSA target uses the action actually taken next
        target = r + gamma * Q(s', a_next)
        Q(s,a) = Q(s,a) + alpha * (target - Q(s,a))

        s = s'
        a = a_next
    until terminal
```

## 8.3 Double Q-learning（tabular）伪代码（交替更新）
```text
Initialize Q_A(s,a), Q_B(s,a) arbitrarily
for episode = 1..:
    s = reset()
    repeat:
        # behavior can be epsilon-greedy w.r.t. (Q_A + Q_B)
        a = epsilon_greedy(sum(Q_A + Q_B), s) 

        take action a, observe r, s'

        with prob 0.5:
            # update Q_A, evaluate with Q_B
            a_star = argmax_{a'} Q_A(s', a')
            target = r + gamma * Q_B(s', a_star)
            Q_A(s,a) = Q_A(s,a) + alpha * (target - Q_A(s,a))
        else:
            # update Q_B, evaluate with Q_A
            a_star = argmax_{a'} Q_B(s', a')
            target = r + gamma * Q_A(s', a_star)
            Q_B(s,a) = Q_B(s,a) + alpha * (target - Q_B(s,a))

        s = s'
    until terminal
```

---

## 9. 记忆卡片（只背这 5 行就够用）

- Q-learning：target 用 $\max$ ⇒ **off-policy**，更激进，可能 overestimate  
- SARSA：target 用实际 $a_{t+1}$ ⇒ **on-policy**，更保守  
- 两者都有 **bootstrap bias**；Q-learning 额外常见 **max overestimation bias**  
- Double Q：**选动作与评估动作分离** ⇒ 降低过估计  
- GLIE：**无限探索 + 最终变贪心**（$\epsilon_t\to0$）

---

所以说就是 q learning的话 采样的时候还是会遵循 epilson greedy采样，但是在更新q表的时候，也是更新的这个动作的q值，但是在更新的时候，qst+1 用的是argmax的方式取的，可能不是下一步的真实action，而 sarsa就是选的就是下一个真实的动作去更新当前动作的q值
