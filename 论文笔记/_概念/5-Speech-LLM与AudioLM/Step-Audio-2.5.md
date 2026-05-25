---
type: concept
aliases: [Step-Audio-2.5, StepAudio2.5, StepAudio-2.5]
---

# Step-Audio-2.5

## 定义

阶跃星辰（StepFun）推出的**统一 audio-language 基础模型**，在 [[Step-Audio-2]] 基础上大幅升级，用一个 MoE LLM backbone 同时覆盖 ASR、TTS 和 Realtime 对话三个方向。

## 核心要点

1. 非对称架构：冻结 audio encoder + 轻量 adaptor + MoE LLM decoder，2.2T token 渐进式预训练。
2. ASR: MTP-5 可验证多 token 解码，RTF **0.0053**（比 [[Qwen3-ASR]]-1.7B 快 1.8x），AISHELL-1 CER **0.71%**。
3. TTS: 全 LLM 合成（去掉 encoder-adaptor），[[Generative Reward Model|GRM]]-[[RLHF]] 偏好对齐，Arena 评测 **67.6%** 胜率。
4. Realtime: 渐进式 SFT + million-scale 人格矩阵 + GRM-PPO，主观评测领先竞品 +10.0 分。

## 代表工作

- StepFun-Audio Team, 2026, arXiv:2605.23463

## 评测/常见数字

- 中文 ASR 平均 CER: 2.97%（5 benchmark 均值）
- 英文 ASR 平均 WER: 3.68%（5 benchmark 均值）
- AISHELL-1 CER: 0.71%
- LibriSpeech clean WER: 1.38%
- ASR RTF: 0.0053
- TTS Arena win rate: 67.6%

## 相关概念

- [[Step-Audio-2]]
- [[Step-Audio]]
- [[Speech LLM]]
- [[Multi-Token Prediction]]
- [[Generative Reward Model]]
- [[Qwen3-ASR]]
- [[Kimi-Audio]]
- [[Qwen2.5-Omni]]
