---
type: concept
aliases: [VocalNet]
domain: SpeechLM
tags: [speech-llm, s2s, thinker-talker]
created: 2026-07-02
---

# VocalNet

## 定义
Thinker-Talker 解耦架构的 S2S 模型，将语义推理（Thinker）和语音渲染（Talker）分离为两个模块。Talker 由 Thinker 的完成文本或交接状态驱动生成语音。

## 核心要点
1. 推理与语音生成解耦，Thinker 负责文本推理，Talker 负责语音渲染
2. 支持流式模式（streaming mode），codec 吞吐量高（~216 tok/s on H100）
3. 相对 PRIME-Speech 等冻结骨干方案，Talker 通常由已完成的文本驱动（串行瓶颈），但 codec 效率更高

## 代表工作
- [[PRIME-Speech]]: 与 VocalNet 在效率和模态 gap 上互有优劣（VocalNet codec 吞吐更高但 S2T-S2S gap 更大）

## 相关概念
- [[Moshi]]
- [[KV Cache]]
- [[Multi-Token Prediction]]
