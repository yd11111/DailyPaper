---
type: concept
aliases: [Mask GCT]
---

# MaskGCT

## 定义

香港中文（深圳）2024 年提出的零样本 TTS：基于 **Masked Generative Codec Transformer**，用 mask-then-predict 的 NAR 方式生成离散 codec token，免 duration 预测、免文本-语音对齐。

## 核心要点

1. NAR mask-and-predict（类 MaskGIT 思路）
2. 帧率 50 Hz，多 stage 离散 token
3. 在 SEED 短句上 SIM 接近最佳，但帧率较高

## 代表工作

- [[VibeVoice]]: 短句基线之一（SEED test-zh CER 2.27, SIM 0.774）
- MaskGCT 原论文 (arXiv 2409.00750, Wang et al. 2024)

## 评测/常见数字

- SEED test-zh CER: 2.27 / SIM: 0.774
- SEED test-en WER: 2.62 / SIM: 0.714
- 帧率: 50 Hz

## 相关概念

- [[Duration Predictor]]
- [[Audio Codec]]
