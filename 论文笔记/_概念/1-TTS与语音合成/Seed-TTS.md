---
type: concept
aliases: [SeedTTS, ByteDance Seed-TTS]
---

# Seed-TTS

## 定义

字节 ByteDance Seed 团队 2024 年提出的高质量零样本 TTS 模型族。包含 Seed-TTS_AR（自回归 token + diffusion）和 Seed-TTS_DiT 等变体；同时贡献了广泛使用的 **Seed-TTS-eval** 评测集（test-zh / test-en，从 CommonVoice 抽取）。

## 数学形式

二阶段：codec token AR 预测 + diffusion 还原。

## 核心要点

1. **Seed-TTS-eval** 几乎已成 zero-shot TTS de facto benchmark（约 1k 英文 + 2k 中文）
2. Seed-TTS 自身在 SEED test-zh CER 1.12 / test-en WER 2.25，是同期最强基线之一
3. 闭源（仅论文，模型未开放）

## 代表工作

- [[VibeVoice]]: 用 Seed-TTS test-zh / test-en 评测短句性能；Seed-TTS 自身分数也作 reference baseline
- Seed-TTS 原论文 (arXiv 2406.02430, Anastassiou et al. 2024)

## 评测/常见数字

- test-zh CER: 1.12
- test-zh SIM: 0.796
- test-en WER: 2.25
- test-en SIM: 0.762

## 相关概念

- [[CosyVoice]]
- [[F5-TTS]]
- [[Spark-TTS]]
