---
title: "HierSpeech++: Bridging the Gap between Semantic and Acoustic Representation of Speech by Hierarchical Variational Inference for Zero-shot Speech Synthesis"
method_name: "HierSpeech++"
authors: [Sang-Hoon Lee, Ha-Yeong Choi, Seung-Bin Kim, Seong-Whan Lee]
year: 2023
venue: arXiv
arxiv_id: "2311.12454"
pdf_path: "assets/papers/HierSpeech++.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, tts]
created: 2026-05-22
---

# HierSpeech++: Hierarchical Variational Inference for Zero-Shot TTS

## 📌 一句话

提出**分层变分推断**框架——用 hierarchical VAE 桥接 semantic representation 和 acoustic representation 之间的 gap，支持 zero-shot TTS / VC / 语音超分辨，在无文本场景下也可做 speech-to-speech。

## 🛠 核心方法

**输入 → 输出**: text/speech → semantic → acoustic → waveform（16/24/48kHz）

**架构组件**（按数据流顺序）:
1. **Text-to-Vec**: text → [[w2v-BERT]] semantic representation
2. **Hierarchical Speech Synthesizer**: semantic → acoustic，分层 VAE（粗 → 细）
3. **SpeechSR**: 语音超分辨率模块（16kHz → 24kHz → 48kHz）
4. **Speaker Encoder**: 条件 speaker embedding

**关键创新**: 通过分层 VAE 将 semantic→acoustic 的巨大 gap 分解为多个小步，每步只需建模局部变化，整体质量和训练稳定性显著提升。

## 🖼 架构图

![Figure 1: HierSpeech++ — 分层语音合成 pipeline（text-to-vec → hierarchical synthesizer → SpeechSR）](https://ar5iv.labs.arxiv.org/html/2311.12454/assets/x1.png)

## 📊 关键结果 / 评测

- LibriTTS test-clean: WER 2.07%, speaker similarity 0.872
- 支持 48kHz 高质量输出

## 💡 借鉴意义（一句话）

做 zero-shot TTS / VC 的人参考——HierSpeech++ 的分层 VAE 思路是非 LM 方案中质量最高的之一。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2311.12454
- PDF: [[assets/papers/HierSpeech++.pdf|本地 PDF]]
- 源目录: `TTS-LLM/HierSpeech.pdf`
