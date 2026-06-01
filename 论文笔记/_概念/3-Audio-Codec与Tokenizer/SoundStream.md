---
type: concept
aliases: [soundstream]
---

# SoundStream

## 定义
Google 提出的神经音频编解码器，将音频波形编码为离散 token 序列。使用 [[RVQ]]（Residual Vector Quantization）多层码本进行量化，是现代离散音频 token 方法的开创性工作之一。

## 数学形式

$$
\text{Encoder}: x \in \mathbb{R}^{T} \rightarrow z \in \mathbb{R}^{T' \times D}, \quad \text{RVQ}: z \rightarrow \{c_1, c_2, \ldots, c_Q\}
$$

- 输入: waveform $x$，采样率 24 kHz
- 编码: 连续 latent $z$，帧率可变
- 量化: $Q$ 层 RVQ 码本，码率 3-18 kbps

## 核心要点
1. 与 [[EnCodec]] 并列为离散音频 token 的两大基础 codec
2. 被 AudioLM、[[VALL-E]] 等后续工作广泛采用作为 audio tokenizer
3. 训练使用重建损失 + 对抗损失（多尺度判别器）
4. 支持可变码率（通过选择 RVQ 层数控制）

## 代表工作
- [[SoundStream]]: 原始论文（Zeghidour et al., 2021）—— 精读笔记见 [[论文笔记/3-Audio-Codec与Tokenizer/SoundStream|SoundStream 精读]]
- AudioLM (Google): 使用 SoundStream token 做音频语言建模
- [[VALL-E]]: 使用 [[EnCodec]]（SoundStream 的同期竞品）做零样本 TTS
- [[GLM-TTS]]: 论文中作为离散 TTS 范式的基础引用

## 评测/常见数字
- 码率: 3-18 kbps
- 采样率: 24 kHz
- ViSQOL > 4.0 at 6 kbps

## 相关概念
- [[EnCodec]]
- [[RVQ]]
- [[DAC]]
- [[Acoustic Token]]
