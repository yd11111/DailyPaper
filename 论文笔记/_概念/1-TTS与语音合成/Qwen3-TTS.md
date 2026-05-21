---
type: concept
aliases: [Qwen3 TTS]
---

# Qwen3-TTS

## 定义

阿里通义 Qwen3 系列的 TTS 模型，**1.7B 参数**，训练数据规模 **~5M 小时**（私有），开权重但闭数据。在 [[Seed-TTS-eval]] 上 WER 1.46 / SIM 0.715，是 2025-2026 年开源 TTS 阵营 WER 最低的标杆之一。

## 核心要点

1. **超大数据 + 中等模型**: 5M 小时数据训 1.7B 模型，与 Meta / 字节大数据小模型路线一致
2. **多语种**: 支持中英日韩等多语种 zero-shot
3. **WER 优势**: [[Raon-OpenTTS]] Table 1 显示 Qwen3-TTS WER 全场最低（1.46），但 SIM 略输 Raon-1B（0.715 vs 0.749）
4. **鲁棒性短板**: 在 [[Raon-OpenTTS-Eval]] Wild 场景 WER 79.14，相比 [[CosyVoice 3]]（8.31）和 Raon-1B（5.61）差距明显，说明私有数据可能偏向干净录音

## 代表工作

- 通义千问 Qwen3-TTS 技术报告

## 相关概念

- [[CosyVoice 2]] / [[CosyVoice 3]]: 同阿里出品的 TTS 路线
- [[Raon-OpenTTS]]: 用 Qwen3-TTS 做对比基线
