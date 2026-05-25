---
type: concept
aliases: [Feed-Forward Transformer, FFT, Feed-Forward Transformer Block]
---

# Feed-Forward Transformer

## 定义
FastSpeech 中提出的 Transformer 变体架构，去除 encoder-decoder attention（cross-attention），仅使用 self-attention + 1D Conv 的前馈结构，实现完全并行的序列到序列生成。

## 核心要点
1. 每个 FFT block = Multi-Head Self-Attention + 2-layer 1D Conv + 残差连接 + LayerNorm
2. 用 1D Conv（kernel=3）替代标准 Transformer 的 position-wise FFN，更好捕捉局部时序特征
3. 音素侧和 mel 侧各堆叠 N 个 FFT blocks，中间用 Length Regulator 连接
4. 无 cross-attention 意味着不存在 attention 漂移问题，保证鲁棒性

## 代表工作
- [[FastSpeech]]: 首次提出 FFT 架构
- [[FastSpeech 2]]: 沿用并扩展

## 相关概念
- [[Transformer]]
- [[Self-Attention]]
- [[Layer Normalization]]
- [[FastSpeech]]
