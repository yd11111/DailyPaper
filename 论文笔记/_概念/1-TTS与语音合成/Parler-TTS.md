---
type: concept
aliases: [Parler-TTS, Parler-TTS-mini, parler-tts]
domain: TTS
tags: [tts, prompt-based-tts, controllable-tts, ar-tts, codec-lm-tts]
created: 2026-05-29
last_updated: 2026-05-29
related_maps:
  - "[[TTS-技术路线图]]"
  - "[[TTS-代表模型谱系]]"
---

# Parler-TTS

## 定义

HuggingFace / Lacombe et al. (2024) 提出的开源 prompt-based AR TTS 模型。用**自然语言风格 prompt**（如"a male voice speaking quickly at high pitch with clean quality"）+ 文本内容 → 语音。基于 Lyth & King 2024 "Natural language guidance of high-fidelity text-to-speech with synthetic annotations" 的工作。

## 架构要点

- **Encoder-Decoder 结构**：text encoder（FLAN-T5 base 系，论文未在 Parler-TTS-mini 文档中重述）输出 prompt embedding；AR decoder 自回归生成 [[DAC]] acoustic token，通过 [[Cross-Attention]] 读 prompt。
- **离散音频表示**：用 [[DAC]] RVQ codec（约 9 层 RVQ，44.1 kHz 音频）做 token 化，属于 [[codec-lm-tts]] 路线。
- **训练数据**：大规模合成标注（Lyth & King 用 ASR + 韵律分析自动给音频打"speed/pitch/gender/quality"标签）。

## 核心要点

1. **prompt-conditioned**：风格控制走自然语言，不需要 reference audio（与 [[VALL-E]] 这类 reference-based 路线对照）。
2. **开源**：是同类工作中最易复现的 prompt-based TTS baseline，HuggingFace 直接可用。
3. **静态全局条件**：风格被当成"对整个 utterance 不变的全局上下文"，不支持句内动态变化——这是 [[StyleSelfReferencing]] (2026) 的论文动机来源。

## 代表工作

- Lacombe et al. (2024) Parler-TTS（HuggingFace 实现）
- Lyth & King (2024) Natural language guidance of high-fidelity text-to-speech with synthetic annotations（方法论奠基）
- [[StyleSelfReferencing]] (2026): 在 Parler-TTS-mini 上加 KV-cache swap + sliding-window 做训练-free 句内风格切换；首次诊断该模型的"style self-referencing"现象

## 局限

- 离散粗粒度控制词（"fast" vs "slightly fast" vs "very fast"），小改 prompt 不一定带来单调声学变化（见 Korotkova et al. 2024 / [[ControlSpeech]] 2025 评估）。
- 风格固定全局；不支持时变 prompt（"start calmly and become excited" 不可解析）。

## 相关概念

- [[PromptTTS]]
- [[InstructTTS]]
- [[Cross-Attention]]
- [[DAC]]
- [[StyleSelfReferencing]]
