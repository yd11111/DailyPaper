---
title: "YourTTS: Towards Zero-Shot Multi-Speaker TTS and Zero-Shot Voice Conversion for everyone"
method_name: "YourTTS"
authors: [Edresson Casanova, Julian Weber, Christopher Shulby, Arnaldo Candido Junior, Eren Gölge]
year: 2022
venue: ICML 2022
arxiv_id: "2112.02418"
pdf_path: "assets/papers/YourTTS.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, tts, vc]
created: 2026-05-22
---

# YourTTS: Zero-Shot Multi-Speaker TTS & VC

## 📌 一句话

基于 [[VITS]] 的 **zero-shot 多说话人多语言** TTS + VC 系统——在 VITS 架构上加入 speaker encoder 和多语言训练，支持 zero-shot voice cloning 和 voice conversion，是 Coqui TTS 生态的核心模型。

## 🛠 核心方法

**输入 → 输出**: text + speaker reference → waveform / source speech + target speaker → converted speech

**架构组件**（按数据流顺序）:
1. **Text Encoder**: phoneme → hidden representation
2. **Speaker Encoder**: 从 reference audio 提取 speaker embedding（H/ASP model）
3. **Flow-based Decoder**: VITS 的 normalizing flow + HiFi-GAN decoder
4. **Posterior Encoder**: 训练时从 GT mel 提取 latent（VAE 框架）

**关键创新**: 在 VITS 框架上首次实现**跨语言 zero-shot voice cloning**——用多语言数据联合训练，speaker encoder 跨语言共享，不需要目标说话人在目标语言的数据。

## 🖼 架构图

![Figure 1(a): YourTTS 训练流程](https://ar5iv.labs.arxiv.org/html/2112.02418/assets/x1.png)

## 📊 关键结果 / 评测

- VCTK: zero-shot speaker similarity SECS 显著优于 baseline
- 支持英/葡/法多语言

## 💡 借鉴意义（一句话）

做 zero-shot TTS / VC 的人了解——YourTTS 是 VITS-based zero-shot TTS 的代表工作，Coqui/XTTS 的直接前身。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2112.02418
- PDF: [[assets/papers/YourTTS.pdf|本地 PDF]]
- 源目录: `TTS-LLM/YourTTS.pdf`
