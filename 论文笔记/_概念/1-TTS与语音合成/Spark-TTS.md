---
type: concept
aliases: [SparkTTS]
---

# Spark-TTS

## 定义

2025 年 SparkAudio 团队的 LLM-based TTS：用单一 LLM 直接预测 single-stream decoupled speech token（BiCodec 输出），把语义/声学信息解耦到一条单流 token 序列，避免多码本展开。

## 核心要点

1. **BiCodec**：单流 codec，把 content token 与 timbre vector 解耦
2. 单 LLM 端到端预测 token，工程简洁
3. 帧率 50 Hz，比 CosyVoice 2 (25 Hz) 高一倍但仍比 EnCodec 低

## 代表工作

- [[VibeVoice]]: 短句对比基线（test-en WER 1.98、test-zh CER 1.20）
- Spark-TTS 原论文 (arXiv 2503.01710, Wang et al. 2025)

## 评测/常见数字

- SEED test-zh CER: 1.20
- SEED test-en WER: 1.98
- 帧率: 50 Hz

## 相关概念

- [[Audio Codec]]
- [[CosyVoice]]
