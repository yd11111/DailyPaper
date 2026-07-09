---
type: concept
aliases: [SALMONN Omni]
domain: SpeechLM
tags: [speech-llm, full-duplex, native-cdm]
created: 2026-07-09
---

# SALMONN-omni

## 定义

Native 端到端全双工语音语言模型，采用 Channel-Division Multiplexing 在单一参数空间内同时处理输入/输出音频流。属于完全共享参数的 CDM 范式。

## 核心要点

1. Native CDM 路线，无外部 VAD
2. 在 Spoken QA 上 Avg S→S = 38.0%
3. 全参数共享导致模态干扰（知识退化问题）

## 代表工作

- [[Lychee-FD]]: 以其作为 Native CDM 基线，对比分层分离的效果

## 相关概念

- [[Channel-Division Multiplexing]]
- [[梯度冲突]]
- [[Moshi]]
