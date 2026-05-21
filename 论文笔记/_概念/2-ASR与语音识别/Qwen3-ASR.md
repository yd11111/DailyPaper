---
type: concept
aliases: [Qwen3 ASR, Qwen3-ASR-1.7B]
---

# Qwen3-ASR

## 定义

阿里 Qwen3 系列下的开源 ASR 基础模型，三段式结构（speech encoder + aligner + Qwen3 LLM），主流参数规模 1.7B。属于把通用 LLM backbone 接到语音前端的 [[Speech LLM]] 路线。

## 核心要点

1. 在常见 ASR 评测（[[LibriSpeech]]、[[AISHELL-1]]、[[Common Voice]]、[[Fleurs]]）上取得 SOTA / 接近 SOTA 的开源成绩。
2. 三段式：speech encoder 提声学特征 → aligner（adapter）做模态对齐 → Qwen3 LLM 解码文字。
3. 鲁棒性仍有空间：在 [[VOiCES]] rm3、[[NOIZEUS]] 0dB 等强噪声子集上 [[WER]] 显著退化。
4. 是 [[MegaASR]] 的 backbone，也是其主要对照基线。

## 代表工作

- 阿里 Qwen 团队（[[Qwen2.5]] / [[Qwen2.5-Omni]] 系列同源）
- 作为 backbone：[[MegaASR]]

## 评测/常见数字

- CHiME-4 Avg [[WER]]: 5.39
- VOiCES Avg [[WER]]: 8.94
- NOIZEUS Avg [[WER]]: 9.45
- LibriSpeech test clean/other: 1.62/3.40

## 相关概念

- [[Speech LLM]]
- [[ASR]]
- [[Whisper]]
- [[Kimi-Audio]]
- [[Step-Audio-2]]
