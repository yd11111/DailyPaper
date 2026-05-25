---
type: concept
aliases: [离散音频 token, Speech Token, 语音 token]
---

# Discrete Audio Token

## 定义
将连续音频信号离散化为有限码本中的 token 序列，使语音可以像文本一样被语言模型处理。分为语义 token（承载内容）和声学 token（承载音色/细节）两类。

## 核心要点
1. **语义 token**: 通常来自 HuBERT/WavLM k-means 或 ASR 模型 VQ（如 CosyVoice 的 S3 token）
2. **声学 token**: 通常来自 Codec 的 RVQ（EnCodec、SoundStream、DAC）
3. token 化是 LLM-based TTS/Audio LM 的基础
4. 关键参数：帧率（Hz）、码本大小、层数（RVQ）

## 代表工作
- [[CosyVoice]]: Supervised semantic token（有监督语义 token）
- [[VALL-E]]: EnCodec 8 层 RVQ token
- [[AudioLM]]: 三段式 semantic → coarse → fine token

## 相关概念
- [[Semantic Token]]
- [[Acoustic Token]]
- [[RVQ]]
- [[Vector Quantization]]
