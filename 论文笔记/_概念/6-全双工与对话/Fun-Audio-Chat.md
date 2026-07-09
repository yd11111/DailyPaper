---
type: concept
aliases: [FunAudioChat]
domain: Dialogue
tags: [full-duplex, thinker-talker, speech-llm]
created: 2026-07-09
---

# Fun-Audio-Chat

## 定义

Thinker-Talker 全双工语音对话模型：将语义推理（Thinker）和声学生成（Talker）解耦为独立模块，通过多阶段训练实现全双工交互。

## 核心要点

1. 属于 Thinker-Talker 架构路线
2. 优势：语义理解能力保持较好（Avg S→S 38.8%）
3. 劣势：多阶段训练复杂、推理瓶颈来自双模块串行

## 代表工作

- [[Lychee-FD]]: 作为 Thinker-Talker 基线对比

## 相关概念

- [[Channel-Division Multiplexing]]
- [[全双工]]
