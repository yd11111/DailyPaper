---
type: concept
aliases: [FFT Block, Feed-Forward Transformer Block]
---

# Feed-Forward Transformer

## 定义

FastSpeech 系列中使用的基础 Transformer block，由 Multi-Head Self-Attention + 1D Convolution 组成，相比标准 Transformer 用 Conv 替代第二个 FFN 层，更适合序列到序列的语音生成任务。

## 核心要点

1. 结构: Self-Attention → LayerNorm → 1D Conv (kernel=9, filter=1024) → LayerNorm → Dropout
2. 在 FastSpeech 2 中 Encoder 和 Decoder 各堆叠 4 层
3. Hidden dim = 256, 2 attention heads
4. 兼顾全局依赖（Self-Attention）和局部模式（Conv1D）

## 代表工作

- [[FastSpeech]]: 首次提出
- [[FastSpeech2]]: 沿用

## 相关概念

- [[Self-Attention]]
- [[Convolution]]
- [[Layer Normalization]]
