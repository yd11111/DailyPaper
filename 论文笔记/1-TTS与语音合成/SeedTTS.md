---
title: "Seed-TTS: A Family of High-Quality Versatile Speech Generation Models"
method_name: "SeedTTS"
authors: [Seed Team, ByteDance]
year: 2024
venue: arXiv
arxiv_id: "2406.02430"
pdf_path: "assets/papers/SeedTTS.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, tts]
created: 2026-05-22
---

# Seed-TTS: High-Quality Versatile Speech Generation

## 📌 一句话

字节跳动推出的**大规模 TTS 模型族**——AR language model + diffusion transformer 架构，在自然度和说话人相似度上接近人类水平，并提出 Seed-TTS_DiT（纯 diffusion 变体）和自蒸馏 VC 方案。是目前工业级 TTS 的标杆之一。

## 🛠 核心方法

**输入 → 输出**: text + speech prompt → speech tokens → waveform

**架构组件**（按数据流顺序）:
1. **Speech Tokenizer**: 将语音离散化为 token
2. **AR Language Model**: 自回归预测 speech token 序列
3. **Diffusion Transformer (DiT)**: token → 连续 mel/latent → 波形（也有纯 DiT 变体 Seed-TTS_DiT）
4. **Acoustic Vocoder**: latent/mel → waveform

**关键创新**: 通过大规模数据 + 精心设计的 tokenizer + RL 对齐，实现了**几乎无法区分于人类**的语音生成质量，同时提出了 Seed-TTS-eval 标准评测 benchmark。

## 🖼 架构图

![Figure 1: Seed-TTS 推理 pipeline 总览（tokenizer → AR LM → DiT → vocoder）](https://ar5iv.labs.arxiv.org/html/2406.02430/assets/x1.png)

## 📊 关键结果 / 评测

- Seed-TTS-eval en: WER 2.2%, SIM-O 0.796
- Seed-TTS-eval zh: CER 1.2%, SIM-O 0.796
- 人类评估: MOS 接近真人录音

## 💡 借鉴意义（一句话）

做 TTS 的人**必读**——Seed-TTS 是目前公开论文中质量最高的 TTS 系统之一，其 eval benchmark 也成为 zero-shot TTS 的标准评测。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2406.02430
- PDF: [[assets/papers/SeedTTS.pdf|本地 PDF]]
- 源目录: `TTS-LLM/seedTTS.pdf`
