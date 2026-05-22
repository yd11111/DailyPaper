---
title: "Speak, Read and Prompt: High-Fidelity Text-to-Speech with Minimal Supervision"
method_name: "SPEAR-TTS"
authors: [Eugene Kharitonov, Damien Vincent, Zalán Borsos, Matt Sharifi, Raphaël Marinier]
year: 2023
venue: arXiv
arxiv_id: "2302.03540"
pdf_path: "assets/papers/SPEAR-TTS.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, tts, speech-llm]
created: 2026-05-22
---

# SPEAR-TTS: High-Fidelity TTS with Minimal Supervision

## 📌 一句话

Google 提出的**两阶段 TTS**——S₁ "reading"（text → semantic token）+ S₂ "speaking"（semantic → acoustic token），通过 backtranslation 大幅减少对 text-speech 配对数据的依赖（仅需 15 分钟标注数据），是 [[AudioLM]] 到 TTS 的关键桥梁。

## 🛠 核心方法

**输入 → 输出**: text + voice prompt → semantic tokens → acoustic tokens → waveform

**架构组件**（按数据流顺序）:
1. **S₁ (Reading)**: text → semantic tokens（[[w2v-BERT]] k-means），用 backtranslation 减少配对数据需求
2. **S₂ (Speaking)**: semantic → acoustic tokens（[[SoundStream]]），用 example prompting 控制说话人
3. **SoundStream Decoder**: acoustic tokens → waveform

**关键创新**: 用 **backtranslation**（先训反向 ASR 模型，再用大量无标注语音生成伪 text-speech 配对）实现了**极少标注数据**的高质量 TTS——仅 15 分钟配对数据即可。

## 🖼 架构图

![Figure 1: SPEAR-TTS — 两阶段架构 S₁(reading) + S₂(speaking)](https://ar5iv.labs.arxiv.org/html/2302.03540/assets/x1.png)

## 📊 关键结果 / 评测

- 仅 15 分钟标注数据即可做 zero-shot TTS
- 质量接近完全监督的 TTS baseline

## 💡 借鉴意义（一句话）

做 low-resource TTS 的人参考——SPEAR-TTS 的 backtranslation + 两阶段设计展示了如何用极少标注数据做高质量 TTS。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2302.03540
- PDF: [[assets/papers/SPEAR-TTS.pdf|本地 PDF]]
- 源目录: `TTS-LLM/spearTTS.pdf`
