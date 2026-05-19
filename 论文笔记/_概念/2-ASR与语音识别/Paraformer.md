---
type: concept
aliases: []
---

# Paraformer

## 定义

阿里 2022 年的非自回归 (NAR) 端到端 ASR 模型，用并行 Transformer 同时预测所有 token 与 token 数。中文社区普遍当 CER 评测器。

## 核心要点

1. NAR + parallel decode → 推理快
2. 中文 ASR 主流基线
3. [[VibeVoice]] 在 SEED test-zh 上用 Paraformer 算 CER

## 代表工作

- 原论文 (Gao et al. 2022, Interspeech)
- [[VibeVoice]] 中文 CER 评测工具

## 评测/常见数字

- AISHELL-1 CER: ~5%
- 推理速度比 AR 快 ~10×

## 相关概念

- [[ASR]]
- [[Whisper]]
