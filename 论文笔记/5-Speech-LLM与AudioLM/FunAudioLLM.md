---
title: "FunAudioLLM: Voice Understanding and Generation Foundation Models for Natural Interaction Between Humans and LLMs"
method_name: "FunAudioLLM"
authors: [Tongyi Speech Team]
year: 2024
venue: arXiv
arxiv_id: "2407.04051"
pdf_path: "assets/papers/FunAudioLLM.pdf"
library_source: "高德文献库"
source_topic: "SSL"
tags: [classic, speech-llm, tts, asr]
created: 2026-05-22
---

# FunAudioLLM: 通义语音基座全景

## 📌 一句话

阿里通义实验室的**语音基座模型全景**技术报告，统一介绍了 SenseVoice（语音理解）+ [[CosyVoice]]（语音生成）两大模型家族，涵盖 ASR / 语种识别 / 情感检测 / TTS / 语音翻译 / 对话等全栈应用场景。

## 🛠 核心方法

**输入 → 输出**: 语音理解（audio → text/label）+ 语音生成（text → speech）

**架构组件**（按模型家族）:
1. **SenseVoice**: 语音理解基座——ASR + 语种 ID + 情感检测 + 音频事件检测
2. **CosyVoice**: 语音生成基座——LLM-based speech token prediction + flow matching decoder
3. **Supervised Semantic Tokenizer**: 监督训练的语音 tokenizer（而非 k-means），提升 token 的语义质量
4. **应用场景**: S2ST / Emotional Voice Chat / Interactive Podcast / Expressive Audiobook

**关键创新**: 首次把语音理解（SenseVoice）和生成（CosyVoice）统一到同一技术体系下，用共享的 semantic tokenizer 打通两端。

## 🖼 架构图

![Figure 1: FunAudioLLM 全景——SenseVoice (理解) + CosyVoice (生成) 双模型体系](https://ar5iv.labs.arxiv.org/html/2407.04051/assets/img/overview-funaudiollm.png)

## 📊 关键结果 / 评测

- SenseVoice: 多语言 ASR / 情感 / 事件检测多任务 SOTA
- CosyVoice: zero-shot TTS + cross-lingual cloning
- 全部开源（模型 + 代码）

## 💡 借鉴意义（一句话）

做 Speech LLM / TTS 的人**必读**——通义是国内 speech foundation model 布局最完整的团队，这份报告是了解其技术路线的最佳入口。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2407.04051
- PDF: [[assets/papers/FunAudioLLM.pdf|本地 PDF]]
- 源目录: `SSL/tongyi.pdf`
