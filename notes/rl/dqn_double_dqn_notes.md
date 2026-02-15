# DQN & Double DQN 学习笔记（含伪代码）

这份笔记把我们对话里确认过的关键点整理成**可直接对照实现**的形式：
- 核心公式（target / TD error / loss）
- DQN vs Double DQN 的唯一差异
- 为什么需要 Replay 与 Target Network
- 完整训练 pipeline 伪代码（可直接改成 PyTorch/JAX）

> 约定：transition 为 `(s_t, a_t, r_{t+1}, s_{t+1}, d_t)`，其中 `d_t=1` 表示终止。

## 1. Q-learning 的 Bellman 最优方程（DQN 的根）

我们要学习最优动作价值函数：



$$

Q^*(s,a)=\mathbb{E}[\sum_{k=0}^\infty \gamma^k r_{t+1+k}\mid s_t=s,a_t=a]

$$

Bellman optimality：

$$

Q^*(s_t,a_t)=\mathbb{E}\Big[r_{t+1}+\gamma(1-d_t)\max_{a'}Q^*(s_{t+1},a')\Big]

$$

**关键理解**：
- `argmax/max` 出现在**算 target**（用 `s_{t+1}`）
- loss 里拟合的是当前执行过的动作：**`Q(s_t, a_t)`**

## 2. DQN：用神经网络近似 Q 表

DQN 用神经网络表示：

$$

Q_\theta(s,a)\approx Q^*(s,a)

$$

训练时通常维护两套参数（常说两个网络）：
- **online network**：$Q_\theta$（参与梯度更新）
- **target network**：$Q_{\theta^-}$（不反传，只慢更新/拷贝）

### 2.1 行为策略：ε-greedy（采样动作）

采样/交互时通常用 online：



$$

a_t=\begin{cases}
\text{random action} & \text{w.p. }\epsilon\\
\arg\max_a Q_\theta(s_t,a) & \text{w.p. }1-\epsilon
\end{cases}

$$

**注意**：采样得到的$a_t$一定用在 loss 里（不要求它是 max）。
即：我们更新的是$Q_\theta(s_t,a_t)$。

### 2.2 DQN 的 TD target / TD error / loss

**DQN target（max 在 target 网络上做）**：



$$

y_t = r_{t+1}+\gamma(1-d_t)\max_{a'}Q_{\theta^-}(s_{t+1},a')

$$

**TD error**：

$$

\delta_t = y_t - Q_\theta(s_t,a_t)

$$

**loss（MSE）**：

$$

L(\theta)=\mathbb{E}\big[(y_t - Q_\theta(s_t,a_t))^2\big]

$$

**done 处理**：若$d_t=1$，则 $(1-d_t)=0$，因此 $y_t=r_{t+1}$。

## 3. 为什么 DQN 需要 Replay + Target Network

### 3.1 Experience Replay（经验回放）
- **减少时间相关性**：连续交互数据强相关，回放采样让 batch 更接近 i.i.d.
- **提高数据复用/样本效率**：同一条经验可训练多次，梯度更稳

### 3.2 Target Network（慢更新）
如果用同一个网络同时产生 target 且被更新，target 会随梯度更新剧烈变化（**moving target**），训练易不稳定甚至发散。

target 网络通过慢更新来让 target 更平滑。

### 3.3 Target Network 的两种更新方式

1. **Hard update（硬拷贝）**：每隔 N 步



$$

\theta^- \leftarrow \theta

$$

2. **Soft update（Polyak / EMA）**：每步一点点

$$

\theta^- \leftarrow \tau\theta + (1-\tau)\theta^-

$$

> 经验：target 更新太频繁 ≈ target 与 online 同步 → moving target 更严重 → 更不稳定。

## 4. DQN 的典型问题：Overestimation Bias（高估偏差）

DQN target 里有：

$$

\max_{a'}Q_{\theta^-}(s',a')

$$

当 Q 估计带噪声时，取 max 会偏向选到被高估的动作，从而导致 **系统性高估**，进而训练不稳。

## 5. Double DQN：online 选动作，target 评估动作（减轻高估）

Double DQN 的唯一差异在于 **target 的计算**：

1. 用 online 选下一状态的贪心动作：

$$

a^* = \arg\max_{a'}Q_\theta(s_{t+1},a')

$$

2. 用 target 评估该动作的 Q 值：

$$

y_t = r_{t+1}+\gamma(1-d_t)\,Q_{\theta^-}(s_{t+1},a^*)

$$

合并写法：

$$

y_t = r_{t+1}+\gamma(1-d_t)\,Q_{\theta^-}\Big(s_{t+1},\arg\max_{a'}Q_\theta(s_{t+1},a')\Big)

$$

**一句话记忆**：
- DQN：max 在 target 上做
- Double DQN：argmax 在 online 上做，但数值由 target 给

**更新规则不变**：仍然最小化$(y_t - Q_\theta(s_t,a_t))^2$，只更新 online 参数 $\theta$。

## 6. DQN / Double DQN 训练 Pipeline（伪代码）

下面伪代码刻意写成**结构化**，你可以直接映射到 PyTorch/JAX。

```python

# -----------------------------

# PSEUDOCODE: DQN / Double DQN

# -----------------------------

init Q_online(theta)
init Q_target(theta_minus)  # copy from online
theta_minus <- theta

init replay_buffer D
init epsilon schedule

for each episode:
    s <- env.reset()
    while not done:
        # 1) 행동 정책 (epsilon-greedy) using ONLINE
        if rand() < epsilon:
            a <- random_action()
        else:
            a <- argmax_a Q_online(s, a)

        # 2) 环境交互
        s_next, r, done <- env.step(a)

        # 3) 存入 replay
        D.add(s, a, r, s_next, done)
        s <- s_next

        # 4) 采样并更新 (满足 warmup / buffer size / update_freq 等条件)
        if D.size >= batch_size and step % update_freq == 0:
            batch <- D.sample(batch_size)
            (S, A, R, S2, Dn) <- batch

            # ---- compute target y ----
            # DQN:
            #   y = R + gamma*(1-Dn)* max_a' Q_target(S2, a')
            # Double DQN:
            #   a_star = argmax_a' Q_online(S2, a')
            #   y = R + gamma*(1-Dn)* Q_target(S2, a_star)

            if algorithm == 'DQN':
                Y <- R + gamma*(1-Dn) * max_a' Q_target(S2, a')
            else if algorithm == 'DoubleDQN':
                A_star <- argmax_a' Q_online(S2, a')
                Y <- R + gamma*(1-Dn) * Q_target(S2, A_star)

            # ---- loss on CURRENT Q(s,a) ----
            Q_sa <- Q_online(S, A)
            loss <- mean( (Y - Q_sa)^2 )

            # ---- gradient step (update ONLINE only) ----
            theta <- theta - lr * grad_theta(loss)

        # 5) 更新 target network（两种方式选一种）
        if hard_update and step % target_update_period == 0:
            theta_minus <- theta
        if soft_update:
            theta_minus <- tau*theta + (1-tau)*theta_minus

        # 6) epsilon decay
        epsilon <- schedule(epsilon)
```


## 7. 一页速记（你写代码时只看这一段）

- transition：$(s_t,a_t,r_{t+1},s_{t+1},d_t)$
- loss 拟合：$Q_\theta(s_t,a_t)$（**采样动作一定用到**）
- done：$d_t=1 \Rightarrow y=r_{t+1}$**DQN target**：

$$

y=r+\gamma(1-d)\max_{a'}Q_{\theta^-}(s',a')

$$

**Double DQN target**：

$$

y=r+\gamma(1-d)Q_{\theta^-}(s',\arg\max_{a'}Q_\theta(s',a'))

$$

**更新**：只更新 online 的$\theta$；target 通过 hard/soft 从 online 慢更新。
