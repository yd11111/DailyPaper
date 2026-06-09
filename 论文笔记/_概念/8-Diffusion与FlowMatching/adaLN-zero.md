---
type: concept
aliases: [Adaptive Layer Norm Zero, 自适应层归一化零初始化]
---

# adaLN-zero

## 定义
Adaptive Layer Normalization with Zero Initialization (adaLN-zero)，[[DiT]] 中的条件注入机制。将外部条件（如 diffusion timestep、class embedding、speaker embedding）通过 MLP 映射为 scale ($\gamma$)、shift ($\beta$)、gate ($\alpha$) 参数，对 [[LayerNorm]] 后的特征做仿射变换，并在初始化时将 gate 设为零，使模型初始行为等价于无条件。

## 核心要点
1. 比 cross-attention 条件注入更高效（无需额外注意力计算）
2. 零初始化确保训练初期不破坏预训练特征
3. 可同时注入多种条件（timestep + speaker + 其他）

## 代表工作
- [[DiT]]: 原始提出
- [[dots-tts]]: AR-FM Head 中用 adaLN-zero 注入 timestep + speaker x-vector
- [[NaturalSpeech3]]: 用于 DiT-based TTS

## 相关概念
- [[DiT]]
- [[Conditional Layer Normalization]]
- [[Classifier-Free Guidance]]
