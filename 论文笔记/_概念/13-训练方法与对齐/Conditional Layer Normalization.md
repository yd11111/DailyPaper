---
type: concept
aliases: [Conditional LayerNorm, CLN, AdaLN, 条件层归一化]
---

# Conditional Layer Normalization

## 定义

将外部条件信息（如说话人 embedding、扩散时间步、风格向量等）通过仿射变换注入 Layer Normalization 的 scale 和 shift 参数中，使归一化后的特征受条件信号调制。

## 数学形式

$$
\text{CLN}(\mathbf{x}, \mathbf{c}) = \gamma(\mathbf{c}) \cdot \frac{\mathbf{x} - \mu}{\sigma} + \beta(\mathbf{c})
$$

- $\mathbf{x}$: 输入特征
- $\mathbf{c}$: 条件向量（speaker embedding / diffusion timestep / style code）
- $\gamma(\mathbf{c}), \beta(\mathbf{c})$: 由条件向量通过线性层或 MLP 生成的 scale 和 shift

## 核心要点
1. 相比直接拼接条件到输入，CLN 通过乘性调制提供更强的控制力
2. 参数效率高——仅增加一个小 MLP 即可注入条件
3. 广泛应用于 Diffusion/Flow TTS（注入时间步）和多说话人 TTS（注入说话人向量）

## 代表工作
- [[NaturalSpeech3]]: 用 CLN 在 FACodec Decoder 中融合 timbre 向量，在 Diffusion 模块中注入时间步
- [[Grad-TTS]]: 用 CLN 注入扩散时间步
- [[DiT]]: Diffusion Transformer 中的 AdaLN

## 相关概念
- [[Layer Normalization]]
- [[FiLM]]
- [[Adaptive Layer Normalization]]
