---
type: concept
aliases: [Smooth Length Reward, 平滑长度奖励]
---

# Smooth Length Reward

## 定义

[[MiniCPM-o 4.5]] 在 [[GRPO]] 中引入的辅助 length reward，用平滑 shaping 取代激进 penalty 风格（如 [[Kimi K1.5]]），在显著压缩响应长度的同时保住 / 提升 task accuracy。

## 数学形式

$$
r_{\mathrm{len}}(i) = \begin{cases} s_{i}, & r_{i}=1 \\ \min(0,\, s_{i}), & r_{i}=0 \end{cases},\quad
s_{i} = \left(0.5 - \frac{\ell_{i}-\ell_{\min}}{\ell_{\max}-\ell_{\min}}\right) \times \min\!\left(1,\, \frac{\ell_{\max}-\ell_{\min}}{\tau}\right)
$$

## 核心要点

1. **正确性条件**: $r_{i}=1$ 时直接取 $s_{i}$；$r_{i}=0$ 时取 $\min(0, s_{i})$ —— 短的错误回答不奖励，避免模型钻空子用短错答。
2. **长度差距缩水项**: $\min(1, (\ell_{\max}-\ell_{\min})/\tau)$ 当同 prompt 内 length spread 很小时整体奖励缩小，减少噪声。
3. **训练稳定性**: 训练曲线接近 no-length-reward baseline，避免 K1.5-style 后期 accuracy 下降。
4. **延迟启用**: 前 480 steps 不计 length reward，等收敛再开。
5. **效果**: thinking 模式下 -35.3% 长度 + benchmark avg +0.8（73.5 → 74.3）。

## 对比

| 方法 | 长度压缩 | Avg 性能变化 | 训练稳定性 |
|---|---|---|---|
| No Length Reward | 0% | 73.5 (baseline) | 平稳 |
| [[Kimi K1.5]] Style | -50.7% (thinking) | 73.5→73.0 (-0.5) | 后期下滑 |
| **Ours (smooth)** | -35.3% (thinking) | 73.5→74.3 (+0.8) | 接近 baseline |

## 代表工作

- [[MiniCPM-o 4.5]]: 提出并消融

## 相关概念

- [[GRPO]]
- [[Kimi K1.5]]
- [[RLHF]]
