---
type: concept
aliases: [Voices Obscured in Complex Environmental Settings, VOiCES corpus]
---

# VOiCES

## 定义

"Voices Obscured in Complex Environmental Settings" 鲁棒 ASR 评测数据集。重放朗读语音到 4 个不同房间（rm1–rm4），叠加 babble / music / none 三种背景，并通过近场（clo）和远场（far）麦克录制。

## 核心要点

1. 多房间 × 多噪声 × 近/远场的多维度组合。
2. 真实物理重放录制，远场场景特别有挑战性。
3. 大规模（~100 万样本量级），常用于评估 [[ASR]] 模型的实际部署鲁棒性。
4. rm3、rm4 是最难的房间（更大、更多反射）。

## 代表工作

- 原始 VOiCES 论文（Richey et al. 2018）
- 评测代表：[[Whisper]]、[[Qwen3-ASR]]、[[MegaASR]]

## 评测/常见数字

- 规模：~1M 条
- [[Whisper]]-L-v3 Avg [[WER]]: 11.79
- [[Qwen3-ASR]] Avg [[WER]]: 8.94
- [[MegaASR]] Avg [[WER]]: 7.35

## 相关概念

- [[CHiME-4]]
- [[NOIZEUS]]
- [[ASR]]
- [[WER]]
