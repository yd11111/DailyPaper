---
title: "CosyVoice: A Scalable Multilingual Zero-shot Text-to-speech Synthesizer based on Supervised Semantic Tokens"
method_name: "CosyVoice"
authors: [Zhihao Du, Qian Chen, Shiliang Zhang, Kai Hu, Heng Lu]
year: 2024
venue: arXiv
arxiv_id: null
pdf_path: "assets/papers/CosyVoice.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, tts]
created: 2026-05-22
---

# CosyVoice: Multilingual Zero-Shot TTS with Supervised Semantic Tokens

## 📌 一句话

阿里通义实验室推出的 zero-shot TTS——核心创新是使用 **supervised semantic tokens**（从 ASR 模型蒸馏的有监督语义 token，而非无监督 SSL token），让 LLM 预测更干净的语义表示，再用 flow matching 生成 mel。

## 🛠 核心方法

**输入 → 输出**: text + speech prompt → semantic tokens → mel → waveform

**架构组件**（按数据流顺序）:
1. **Supervised Speech Tokenizer**: 基于 ASR encoder 的有监督语义 tokenizer（区别于 HuBERT k-means）
2. **LLM (AR)**: text + prompt → semantic token 序列
3. **Flow Matching**: semantic token → mel-spectrogram
4. **HiFi-GAN Vocoder**: mel → waveform

**关键创新**: 提出 **supervised semantic token**——用 ASR 模型（而非 SSL 模型）训练 tokenizer，token 对应真实语言单元（类似 phoneme），比无监督 token 语义更干净。

## 📊 关键结果 / 评测

- 支持中/英/日/粤/韩多语言
- 完全开源（模型 + 训练代码）

## 💡 借鉴意义（一句话）

做 TTS / Speech LLM 的人关注——CosyVoice 的 supervised semantic token 思路指出了一条与 SSL token 不同的 tokenization 路线。

## 🔗 链接

- PDF: [[assets/papers/CosyVoice.pdf|本地 PDF]]
- 源目录: `TTS-LLM/CosyVoice_v1.pdf`
