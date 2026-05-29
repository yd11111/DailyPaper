---
type: concept
aliases: [FunASR-Nano, Fun-ASR-Nano, Fun-ASR-Nano-2512]
domain: ASR
tags: [asr, evaluation-tool, multilingual, lightweight]
related_maps:
  - "[[ASR-技术路线图]]"
created: 2026-05-29
last_updated: 2026-05-29
---

# FunASR-Nano

## 定义

阿里 FunAudioLLM 团队 2025-12 发布的轻量级 ASR 模型，专为评测 / 工程 pipeline 设计。HF 路径：<https://huggingface.co/FunAudioLLM/Fun-ASR-Nano-2512>

## 核心要点

1. **性能接近参数量级 SOTA**：[[SwanBench-Speech]] §C.4 报告 LibriSpeech-clean WER **1.76%**、Fleurs-zh CER **2.56%**——与同 scale 主流模型相当（参考 Open ASR Leaderboard, Srivastav et al. 2025）
2. **小模型 / 快推理**：适合大批量评测场景；在 [[SwanBench-Speech]] 中作为 Content Accuracy 评测的转写模型
3. 与 [[Whisper]]-large-v3 / [[Paraformer]] / [[SenseVoice]] 同属 FunAudioLLM 系列工具链
4. 仓库：<https://github.com/FunAudioLLM/Fun-ASR>

## 代表工作

- [[SwanBench-Speech]] 用作长篇 TTS 评测的 ASR 转写模型
- FunAudioLLM 工具链一员

## 相关概念

- [[Whisper]]
- [[SenseVoice]]
- [[Paraformer]]
- [[WER]] / [[CER]]
- [[Content Accuracy]]
