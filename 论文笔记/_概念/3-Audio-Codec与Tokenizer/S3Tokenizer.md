---
type: concept
aliases: [S3 Tokenizer, Supervised Semantic Speech Tokenizer]
domain: Codec
tags: [speech-tokenizer, semantic-token, cosyvoice]
created: 2026-05-29
last_updated: 2026-05-29
related_maps:
  - "[[TTS-表示层地图]]"
---

# S3Tokenizer

## 定义
CosyVoice 系列使用的**有监督语义语音 tokenizer**，把波形编码为低帧率离散**语义 token**（[[Semantic Token]]，非 [[Audio Codec|声学 RVQ token]]）。通过在 ASR 模型（SenseVoice-Large 类）中间插入向量量化层、用 ASR loss 监督训练得到，使 token 偏向承载语言内容。v2 为 25 Hz。

## 数学形式
$\text{waveform} \rightarrow \text{S3Tokenizer} \rightarrow \mathbf{c}\in\{0,\ldots,N-1\}^{T}$，帧率 25 Hz（v2），下采样长度 $T$。

## 核心要点
1. 语义导向——token 主要编码内容/发音，说话人/音色信息相对弱，需配 detokenizer（CFM+vocoder）补声学细节。
2. 低帧率（25 Hz）适合 LM 建模，序列短。
3. v1 / v2 两代，[[CosyVoice]] 用 v1、[[CosyVoice 2]] 用 v2。

## 代表工作
- [[CosyVoice]] / [[CosyVoice 2]]: 原生使用。
- [[PALLE]]: 用 S3Tokenizer v2 25Hz 作语音表示，配 CosyVoice 2 detokenizer。

## 评测/常见数字
[[PALLE]] Tab.1：GT 经 S3Tokenizer v2 重建后 cross-sentence WER-H 3.51、SIM-o 0.743（即语义 token 路线的重建上限）。

## 相关概念
- [[Semantic Token]]
- [[CosyVoice 2]]
- [[Conditional Flow Matching]]
