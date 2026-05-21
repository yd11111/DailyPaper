---
type: concept
aliases: [Step-Audio-2, Step-Audio-2-mini, StepAudio2]
---

# Step-Audio-2

## 定义

阶跃星辰开源的端到端 [[Speech LLM]]，前代为 Step-Audio。具备 ASR、TTS、语音对话等多任务能力，提供 mini（轻量）与全量两个尺寸。

## 核心要点

1. 端到端音频理解+生成统一架构。
2. ASR 子任务上表现强：在干净 benchmark 与多数噪声 benchmark 接近 [[Qwen3-ASR]] 水平。
3. Step-Audio-2-mini 是该系列轻量版本，[[LibriSpeech]] test clean [[WER]] 1.37。

## 代表工作

- 阶跃星辰（StepFun）2025–2026 系列

## 评测/常见数字

- CHiME-4 Avg [[WER]]: 6.20（mini）
- VOiCES Avg [[WER]]: 10.56（mini）
- LibriSpeech test clean/other [[WER]]: 1.37 / 2.75（mini）

## 相关概念

- [[Speech LLM]]
- [[Kimi-Audio]]
- [[Qwen2.5-Omni]]
- [[Qwen3-ASR]]
