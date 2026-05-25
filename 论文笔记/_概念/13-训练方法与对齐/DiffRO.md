---
type: concept
aliases: [Differentiable Reward Optimization, 可微分奖励优化]
---

# DiffRO (Differentiable Reward Optimization)

## 定义

一种用于 LLM-based TTS 的后训练方法，通过 Gumbel-Softmax 重参数化使 LLM 输出的离散 speech token 采样可微分，直接在 token 空间用 ASR 后验概率等作为奖励信号进行反向传播优化，绕过 CFM 和 Vocoder 的前向计算。

## 数学形式

$$
\pi_\theta^* = \max_{\pi_\theta} \mathbb{E}[R(Y)] - \beta D_{KL}[\pi_\theta(\mu|Y) \| \pi_{ref}(\mu|Y)]
$$

奖励函数（ASR reward）：

$$
R_{ASR}(Y) = \log P_{ASR}(\tilde{Y}_n = Y_n | Y_{1:n-1}; \tilde{\mu}_{1:T})
$$

KL 散度在 token-level logits 上计算，而非序列级。

## 核心要点

1. 关键创新：用 Gumbel-Softmax 使离散 token 采样可微，绕过 CFM/Vocoder 的高计算开销
2. Token2Text 模型作为奖励：在 speech token 空间直接评估内容一致性
3. 支持多任务奖励扩展（ASR + SER + MOS 预测 + AED 等）
4. 存在 reward hacking 问题：提升内容一致性但可能降低说话人相似度
5. 通用性：理论上可应用于任何 LLM-based TTS 系统

## 代表工作

- [[CosyVoice3]]: 提出 DiffRO，实现 20-50% 相对 WER 提升
- F5R-TTS: 使用 GRPO 做 TTS RL（对比方法，需完整 CFM+Vocoder 前向）

## 评测/常见数字

- SEED-TTS-eval: DiffRO 贡献 12-35% 相对 CER/WER 提升
- 跨语言场景半数条件 >50% 相对提升
- 韩语最高达 68.6% 相对 WER 提升

## 相关概念

- [[Gumbel-Softmax]]
- [[Speech DPO]]
- [[GRPO]]
- [[CosyVoice3]]
