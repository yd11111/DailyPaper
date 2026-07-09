---
type: concept
aliases: [Step-Audio, StepAudio-2, StepAudio-2-mini, 阶跃星辰语音模型]
domain: SpeechLM
tags: [speech-llm, full-duplex, industrial-system]
created: 2026-07-09
---

# StepAudio

## 定义

阶跃星辰（StepFun）开源的多模态语音模型系列。StepAudio-2-mini 是约 10B 参数的半双工 SLM，支持语音对话和指令跟随。被 Lychee-FD 用作 warm-start backbone。

## 核心要点

1. 阶跃星辰出品，属于工业级 Speech LLM
2. StepAudio-2-mini 在 Spoken QA 上 Avg S→T = 51.3%，Avg S→S = 40.9%
3. 作为强半双工 baseline，验证 Lychee-FD 全双工适配后能否保持/超越其能力

## 代表工作

- [[Lychee-FD]]: 以 StepAudio-2-mini 为 backbone 做全双工适配

## 相关概念

- [[CosyVoice]]
- [[Whisper]]
- [[全双工]]
