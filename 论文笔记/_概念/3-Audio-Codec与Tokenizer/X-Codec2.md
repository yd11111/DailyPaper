---
type: concept
aliases: [XCodec2, X-Codec 2]
---

# X-Codec2

## 定义

面向语音的单层量化 codec，使用 Finite Scalar Quantization (FSQ) 替代 RVQ，在 50Hz 帧率下以单层码本大小 65,536 实现高质量语音编解码。专为 LLM-based TTS 设计，因为单层量化意味着每秒只需预测 50 个 token（低序列长度），对 LLM 自回归生成友好。

## 核心要点
1. 50Hz 帧率，单层 FSQ，码本 65,536 → 50 token/s
2. 相比 RVQ-based codec（如 EnCodec 8层），序列长度缩短 4-8x
3. 与 [[X-Codec]]（通用音频 RVQ codec）配合使用，分别处理 speech / non-speech
4. 被 [[Audex]] 用作语音 codec

## 代表工作
- [[Audex]]: 用 X-Codec2 做语音 tokenization
- Ye et al., 2025b: X-Codec2 原始论文

## 评测/常见数字
- 16kHz 语音 → 50 token/s（单层）
- 相比 EnCodec/SoundStream 的 50Hz×8层=400 token/s，大幅降低

## 相关概念
- [[X-Codec]]
- [[FSQ]]
- [[RVQ]]
- [[EnCodec]]
