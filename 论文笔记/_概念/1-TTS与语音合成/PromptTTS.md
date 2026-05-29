---
type: concept
aliases: [PromptTTS, Prompt-based TTS]
domain: TTS
tags: [tts, prompt-based-tts, controllable-tts, instruction-tts]
created: 2026-05-29
last_updated: 2026-05-29
related_maps:
  - "[[TTS-技术路线图]]"
---

# PromptTTS

## 定义

Guo et al. (2023) "PromptTTS: controllable text-to-speech with text descriptions" 提出的**奠基性提示式 TTS**：学习从文字描述（speech style description）到声学风格 latent 的映射，从而通过自然语言 prompt 控制属性（gender / pitch / speed / emotion / quality 等）。

## 核心要点

1. **首个把 TTS 控制接口从 reference audio 切换到自然语言**的工作之一，开创了 prompt-based TTS 路线。
2. 设计了"style description 编码器 + content + style 双通道"的基本范式，被后续 [[InstructTTS]] / [[Parler-TTS]] / [[ControlSpeech]] 继承和扩展。
3. 接受**粗粒度离散词汇**（如 "fast"/"slightly fast"/"very fast"），不支持连续插值——这是后续 [[StyleSelfReferencing]] (2026) 等训练-free 推理控制工作的动机来源之一。

## 代表工作

- Guo et al. (2023) PromptTTS — 范式奠基
- Leng et al. (2023) PromptTTS 2 — 引入描述生成 + 改进可控性
- [[InstructTTS]] (Yang et al. 2024) — 跨模态度量学习 + 自由格式指令
- [[Parler-TTS]] (Lacombe et al. 2024) — 开源 + 大规模合成标注训练
- [[ControlSpeech]] (Ji et al. 2025) — 同时做 zero-shot 克隆 + zero-shot 风格控制
- [[StyleSelfReferencing]] (2026) — 训练-free 推理时细粒度控制

## 局限（共同问题）

- prompt 编辑 → 声学变化的**单调性 / 可预测性**弱
- 风格被当成静态全局条件，不支持时变 trajectory
- 不同实现间缺 community-agreed benchmark

## 相关概念

- [[InstructTTS]]
- [[Parler-TTS]]
- [[ControlSpeech]]
- [[StyleSelfReferencing]]
- [[Style Self-Referencing]]
