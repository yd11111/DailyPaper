---
type: concept
aliases: [LLaSA, LLASA]
---

# Llasa

## 定义

由 Ye et al. (2025) 提出的 LLaMA-based scalable speech synthesis 系统：把 TTS 当作 LLaMA 风格自回归语言建模任务，在 train-time + inference-time 都按 LLM scaling laws 推。配套提出一个改良的离散 speech codec（含 [[MFD]] adversarial 训练）。

## 核心要点

1. **LLM-native TTS**：单一 LLaMA backbone 直接预测 codec token 序列，无需独立 acoustic model。
2. **train-time + inference-time scaling**：同时探索两端的算力 → 性能曲线。
3. **配套 codec 贡献**：引入 [[MFD]] (Multi-Frequency Discriminator) 提高 codec 质量。

## 代表工作

- Ye et al. (2025): "Llasa: scaling train-time and inference-time compute for llama-based speech synthesis" (arXiv)。
- 被 [[LoSATok]] 借用其 MFD discriminator 设计。

## 相关概念

- [[VALL-E]] / [[CosyVoice]]：同为 codec + LM 范式。
- [[MFD]]：本工作引入的判别器，被广泛复用。
- [[Speech LLM]] / [[Codec LM]]
