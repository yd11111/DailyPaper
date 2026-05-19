---
type: concept
aliases: [GRPO, Group Relative Policy Optimization]
---

# GRPO (Group Relative Policy Optimization)

## 定义

DeepSeek 在 [DeepSeekMath] 中提出的强化学习算法，用 **同一 prompt 下多条 sample 的相对排序** 替代 [[PPO]] 中的 critic value head，省去 value function 训练。是 [[Reasoning RL]] 主流算法。

## 数学形式（直观）

对同一 prompt 采样 $G$ 条 response $\{o_1, ..., o_G\}$，得 reward $\{r_1, ..., r_G\}$。Advantage 用 group 内归一化：

$$
A_i = \frac{r_i - \mu_r}{\sigma_r}
$$

策略目标：

$$
\mathcal{J}_{\text{GRPO}} = \mathbb{E}\left[\frac{1}{G}\sum_i \frac{\pi_\theta(o_i|q)}{\pi_{\theta_{\text{old}}}(o_i|q)} A_i\right] - \beta \, D_{\text{KL}}(\pi_\theta \| \pi_{\text{ref}})
$$

## 核心要点

1. **无 value model**：节省内存与训练成本
2. **Group baseline**: 同 prompt 下 reward 相对排序作 advantage
3. **可叠加多种 reward**: rule-based 验证 / judge model / format / length 等
4. **被广泛用于 reasoning RL**: DeepSeek-R1, Kimi K1.5, MiniCPM-o 4.5 等

## 代表工作

- [[DeepSeekMath]]: 提出
- [[DeepSeek-R1]]
- [[Kimi K1.5]]
- [[MiniCPM-o 4.5]]: 配 [[Smooth Length Reward]] 和 [[RLAIF-V]] 使用

## 相关概念

- [[PPO]]
- [[DPO]]
- [[RLAIF-V]]
- [[Smooth Length Reward]]
