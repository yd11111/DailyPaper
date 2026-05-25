---
type: concept
aliases: [Byte Pair Encoding, 字节对编码]
---

# BPE

## 定义
Byte Pair Encoding，一种子词分词算法。通过迭代合并最频繁的字节/字符对构建词表，广泛用于 NLP 和 TTS 的文本编码。

## 核心要点
1. 自动发现子词单元，平衡词表大小与覆盖率
2. CosyVoice 用 BPE 替代 phoneme 作为文本输入，避免依赖外部 phonemizer
3. SentencePiece 是常用的 BPE 实现

## 代表工作
- [[CosyVoice]]: 用 BPE 替代 phoneme，WER 进一步降低（5.05% → 3.93%）

## 相关概念
- [[Phoneme]]
- [[Transformer]]
