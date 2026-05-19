---
type: concept
aliases: [Perceptual Evaluation of Speech Quality]
---

# PESQ

## 定义

ITU-T P.862 标准：基于感知模型的客观语音质量评分（1–4.5），将参考 + 失真信号比对得到 MOS-LQO。最初为电信编解码器评测设计，沿用至 codec 评测。

## 核心要点

1. 需要 reference + distorted 两条信号
2. 高于 STOI 关注感知相似度
3. [[VibeVoice]] tokenizer 在 LibriTTS test-clean PESQ 3.068（最高）

## 代表工作

- 原 paper (Rix et al. ICASSP 2001)
- [[VibeVoice]]、[[EnCodec]]、[[DAC]] 等几乎所有 codec 评测都用

## 评测/常见数字

- 强 codec: 2.5–3.0 (低码率) / 3.5–4.0 (高码率)
- [[VibeVoice]] tokenizer 7.5 Hz: 3.068 (test-clean) / 2.848 (test-other)

## 相关概念

- [[STOI]]
- [[UTMOS]]
- [[Audio Codec]]
