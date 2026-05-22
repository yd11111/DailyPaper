---
title: "GLM-TTS Technical Report"
method_name: "GLM-TTS"
authors: [Jiayan Cui, Zhihan Yang, Naihan Li, Jiankun Tian, Xingyu Ma]
year: 2024
venue: arXiv
arxiv_id: "2512.14291"
pdf_path: "assets/papers/GLM-TTS.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, tts]
created: 2026-05-22
---

# GLM-TTS: Zhipu AI TTS Technical Report

## 📌 一句话

智谱 AI 推出的 TTS 系统——基于 GLM 架构的语音合成，强调在中文场景下的自然度和表现力，工业级系统。

## 🛠 核心方法

**输入 → 输出**: text → speech

**架构组件**:
1. **GLM Backbone**: 基于 GLM 的语言模型架构
2. **Speech Tokenizer**: 语音离散化
3. **Speech Decoder**: token → waveform

**关键创新**: 将 GLM（General Language Model）架构应用于 TTS，结合智谱的 LLM 技术栈。

## 📊 关键结果 / 评测

- 中文 TTS 高自然度
- 在线 demo: audio.z.ai

## 💡 借鉴意义（一句话）

做中文 TTS 的人了解——GLM-TTS 是国内大模型团队在 TTS 领域的代表性工作。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2512.14291
- PDF: [[assets/papers/GLM-TTS.pdf|本地 PDF]]
- 源目录: `TTS-LLM/GLM-TTS.pdf`
