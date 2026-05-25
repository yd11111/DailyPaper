---
type: concept
aliases: [Gumbel Softmax, Gumbel-Softmax Trick, 重参数化采样]
---

# Gumbel-Softmax

## 定义

一种重参数化技巧，使从离散分类分布的采样过程可微分。通过添加 Gumbel 噪声并用 Softmax（温度参数 $\tau$）近似 argmax，生成可微的 one-hot 或 soft token 表示。

## 数学形式

$$
y_i = \frac{\exp((\log \pi_i + g_i) / \tau)}{\sum_j \exp((\log \pi_j + g_j) / \tau)}
$$

其中 $g_i \sim \operatorname{Gumbel}(0, 1)$，$\tau \to 0$ 时趋近于真正的 one-hot 采样。

## 核心要点

1. 解决离散变量无法反向传播的问题
2. 温度 $\tau$ 控制近似程度：高温 → 均匀分布，低温 → 接近 argmax
3. 常与 Straight-Through Estimator 结合使用
4. 在 TTS 领域被 CosyVoice 3 的 DiffRO 用于使 speech token 采样可微

## 代表工作

- Jang et al. 2017: "Categorical reparameterization with Gumbel-Softmax"
- [[CosyVoice3]]: 在 DiffRO 中用 Gumbel-Softmax 使 LLM speech token 采样可微

## 相关概念

- [[DiffRO]]
- [[VAE]]
