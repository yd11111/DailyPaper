---
type: concept
aliases: [InstructTTS]
domain: TTS
tags: [tts, prompt-based-tts, controllable-tts, instruction-tts]
created: 2026-05-29
last_updated: 2026-05-29
related_maps:
  - "[[TTS-技术路线图]]"
---

# InstructTTS

## 定义

Yang et al. (2024) "InstructTTS: Modelling Expressive TTS in Discrete Latent Space with Natural Language Style Prompt" — 在离散 latent 空间用**跨模态度量学习**（cross-modal metric learning）建立"自然语言指令 ↔ 声学风格"对齐，使 TTS 能跟随自由形式的风格指令。

## 核心要点

1. 相对 [[PromptTTS]] 的进步：从"学一个映射模型"升级为"显式学一个跨模态对齐空间"，理论上支持更自由格式的描述。
2. 风格表示在**离散 latent**（neural codec token 域），与同期 codec-LM TTS（[[VALL-E]]）的离散化趋势一致。
3. 仍然是静态全局风格条件（与 [[Parler-TTS]] / [[PromptTTS]] 同病）。

## 代表工作

- Yang et al. (2024) InstructTTS（原始论文）

## 局限

- 与同期 [[Parler-TTS]] / [[PromptTTS]]-2 类似，prompt 控制的连续性 / 时变性 / 跨样本一致性都不强
- 不开源（相对 [[Parler-TTS]] 的可获取性差很多）

## 相关概念

- [[PromptTTS]]
- [[Parler-TTS]]
- [[ControlSpeech]]
- [[StyleSelfReferencing]]
