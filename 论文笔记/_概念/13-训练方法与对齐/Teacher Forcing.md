---
type: concept
aliases: [Teacher-Forcing, teacher forcing]
---

# Teacher Forcing

## 定义

自回归模型训练时的标准策略：在每一步使用 ground-truth token（而非模型自身生成的 token）作为下一步的输入条件，避免训练时误差累积。

## 数学形式

$$
\mathcal{L} = -\sum_{t=1}^{T} \log p_\theta(y_t \mid y_1^{\text{gt}}, y_2^{\text{gt}}, \dots, y_{t-1}^{\text{gt}})
$$

其中 $y_t^{\text{gt}}$ 是 ground-truth token，$p_\theta$ 是模型预测的条件概率。

## 核心要点

1. 训练效率高：每步都用正确上下文，梯度信号清晰
2. **Exposure Bias**：训练时看 ground-truth，推理时看自身生成，分布不匹配是主要缺陷
3. 常配合 [[Cross Entropy]] 损失使用
4. 缓解 exposure bias 的方法包括 scheduled sampling、对抗训练、强化学习微调

## 代表工作

- [[MegaTTS]]: P-LLM 用 teacher forcing + cross-entropy 训练韵律码预测
- [[VALL-E]]: AR 阶段用 teacher forcing 训练 codec token 预测
- [[Tacotron 2]]: 经典 AR TTS 中 teacher forcing 训练 mel 帧预测

## 相关概念

- [[Autoregressive]]
- [[Cross Entropy]]
- [[Error Accumulation]]
