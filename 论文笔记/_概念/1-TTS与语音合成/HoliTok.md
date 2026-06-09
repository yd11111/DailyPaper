---
type: concept
aliases: [Holistic Tokenizer]
---

# HoliTok

## 定义
一种多目标训练策略，用于构建语义结构化的连续/离散音频表示。核心思想：在 VAE/codec 训练中同时优化重建质量和下游任务性能（ASR、情感、说话人等），使 latent 空间兼具高保真重建和语义可学习性。

## 核心要点
1. 不仅优化重建 loss，还加入 SSL 教师对齐（如 WavLM/HuBERT）和多任务 supervision
2. 使 latent 空间适合后续 LM 建模（prediction-friendly）
3. 训练完成后丢弃下游 head，只保留编码器

## 代表工作
- HoliTok 原始论文
- [[dots-tts]]: AudioVAE 采用 HoliTok 式两阶段训练

## 相关概念
- [[AudioVAE]]
- [[WavLM]]
- [[SpeechTokenizer]]
