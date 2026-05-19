---
type: concept
aliases: [声学 tokenizer]
---

# Acoustic Tokenizer

## 定义

负责"声学还原"的 speech tokenizer——从波形提取保留音色 / 韵律 / 频谱细节的 token，让 decoder 能高保真还原原波形。与 [[Semantic Tokenizer]] 互补（后者只承载语言内容）。

## 核心要点

1. 通常基于 RVQ codec ([[EnCodec]] / [[DAC]]) 或连续 VAE ([[VibeVoice]])
2. 衡量：PESQ、STOI、UTMOS（重建质量）
3. AudioLM / SpeechTokenizer 中常与 Semantic Tokenizer 分层使用
4. **VibeVoice 创新**：连续 σ-VAE 方式，7.5 Hz / 单维 latent

## 代表工作

- [[EnCodec]] / [[DAC]]
- [[VibeVoice]]: 提出 7.5 Hz σ-VAE acoustic tokenizer

## 相关概念

- [[Semantic Tokenizer]]
- [[Audio Codec]]
- [[VAE]]
