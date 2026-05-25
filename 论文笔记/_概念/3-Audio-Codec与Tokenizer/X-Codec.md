---
type: concept
aliases: [X-Codec, X-Codec 2]
---

# X-Codec

## 定义
同时建模语义信息和声学信息的统一音频编解码器，试图在单一 codec 中兼顾 semantic token 的语言判别力和 acoustic token 的重建质量。

## 核心要点
1. 在 RVQ 编码中融入语义监督信号（如 HuBERT/WavLM 特征）
2. 目标：解决 AudioLM 范式中需要两个独立 tokenizer 的问题
3. X-Codec 2 进一步提升语义-声学统一质量

## 代表工作
- [[AudioLM]]: X-Codec 试图统一 AudioLM 中分离的 semantic 和 acoustic tokenizer

## 相关概念
- [[Semantic Token]]
- [[Acoustic Token]]
- [[RVQ]]
- [[SpeechTokenizer]]
- [[EnCodec]]
