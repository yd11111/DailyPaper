---
type: concept
aliases: [Conformer]
---

# Conformer

## 定义

Google 2020 年提出的 ASR 编码器架构 (Gulati et al., Interspeech 2020)。把卷积模块插入 Transformer block 内部（self-attention → conv → FFN 的 macaron 三明治），同时建模局部特征和全局上下文，成为端到端 ASR 的主流 backbone。

## 数学形式

每个 Conformer block：
$$\tilde{x} = x + \tfrac{1}{2}\text{FFN}(x)$$
$$x' = \tilde{x} + \text{MHSA}(\tilde{x})$$
$$x'' = x' + \text{Conv}(x')$$
$$y = \text{LayerNorm}(x'' + \tfrac{1}{2}\text{FFN}(x''))$$

其中 Conv 模块为 pointwise → GLU → depthwise → BN → swish → pointwise。

## 核心要点

1. **macaron 结构**：两个半权 FFN 夹住 MHSA + Conv，比单 FFN 收敛更稳
2. **局部 + 全局结合**：Conv 抓局部音素级模式，MHSA 抓长程语义
3. 在 LibriSpeech test-clean 上 WER 1.9%（带 LM），是当年 SOTA
4. 后续 [[Zipformer]]、[[Squeezeformer]] 都是其变种

## 代表工作

- 原论文 Interspeech 2020 (arXiv 2005.08100)
- [[MedASR]]：Google 105M Conformer 用于医疗 dictation
- [[Whisper]]：编码器侧也采用 Conformer-like 结构

## 评测/常见数字

- LibriSpeech 100M 参数 Conformer：test-clean WER 2.1% / test-other WER 4.3%（不带外部 LM）

## 相关概念

- [[CTC]]
- [[RNN-T]]
- [[Whisper]]
