---
type: concept
aliases: [CHiME4, CHiME-4 Challenge]
---

# CHiME-4

## 定义

经典鲁棒 ASR Challenge 数据集，由 4 种环境（公交 bus / 咖啡馆 caf / 步行 ped / 街道 str）下录制 + 仿真组成，共 16 个细分子集（dt05/et05 × 4 环境 × real/simu）。

## 核心要点

1. 4 种背景环境噪声场景。
2. 同时包含真录（real）和仿真（simu）样本，便于评估仿真→真实泛化。
3. 与 [[NOIZEUS]] / [[VOiCES]] 并列为鲁棒 ASR 的标准 benchmark。
4. 整体难度较低，强模型平均 [[WER]] 5–7%。

## 代表工作

- 原始 CHiME-4 Challenge 论文（Vincent et al.）
- 评测代表：[[Whisper]]、[[Qwen3-ASR]]、[[MegaASR]]

## 评测/常见数字

- 规模：~15K 条
- [[Whisper]]-Large-v3 Avg [[WER]]: 7.02
- [[Qwen3-ASR]] Avg [[WER]]: 5.39
- [[MegaASR]] Avg [[WER]]: 5.00（含 router）

## 相关概念

- [[VOiCES]]
- [[NOIZEUS]]
- [[ASR]]
- [[WER]]
