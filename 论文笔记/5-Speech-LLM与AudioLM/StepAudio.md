---
title: "Step-Audio: Unified Understanding and Generation in Intelligent Speech Interaction"
method_name: "StepAudio"
authors: [Step-Audio Team, StepFun]
year: 2025
venue: arXiv
arxiv_id: "2502.11946"
pdf_path: "assets/papers/StepAudio.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, speech-llm]
created: 2026-05-22
---

# Step-Audio: Unified Speech Understanding & Generation

## 📌 一句话

阶跃星辰推出的**统一语音交互系统**——speech tokenizer + LLM + speech decoder 三合一，支持语音理解和生成，还集成了 tool call / 情感控制 / 实时推理 pipeline。

## 🛠 核心方法

**输入 → 输出**: speech/text → LLM reasoning → speech/text response

**架构组件**（按架构层次）:
1. **Speech Tokenizer**: 语音 → 离散 token
2. **LLM Backbone**: 统一处理 text + speech token 的大语言模型
3. **Speech Decoder**: speech token → waveform
4. **Real-time Pipeline**: VAD + streaming tokenizer + speculative generation

**关键创新**: 完整的**工业级语音交互系统**——不仅有模型，还包括 real-time inference pipeline（VAD / streaming / tool call / emotion control），是开源 Speech LLM 系统中工程完整度最高的之一。

## 🖼 架构图

![Figure 2: Step-Audio 架构——tokenizer + LLM + decoder](https://ar5iv.labs.arxiv.org/html/2502.11946/assets/x1.png)

## 📊 关键结果 / 评测

- ASR: 平均 CER 4.64（6 个 benchmark，优于 Whisper Large-v3 的 7.28）
- TTS: CER 1.31%（中文）/ WER 2.31%（英文），优于 CosyVoice 2 / MaskGCT
- Spoken QA: Llama Question 81.0, ComplexBench 74.0（开源 SOTA）
- 推理加速: speculative generation 减少约 500ms 延迟

## 💡 借鉴意义（一句话）

做 Speech LLM / 语音交互系统的人关注——Step-Audio 的 real-time pipeline 和 tool call 集成值得工程参考。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2502.11946
- PDF: [[assets/papers/StepAudio.pdf|本地 PDF]]
- 源目录: `TTS-LLM/StepAudio.pdf`
