---
title: "NaturalSpeech 2: Latent Diffusion Models are Natural and Zero-Shot Speech and Singing Synthesizers"
method_name: "NaturalSpeech2"
authors: [Kai Shen, Zeqian Ju, Xu Tan, Yanqing Liu, Yichong Leng]
year: 2023
venue: arXiv
arxiv_id: "2304.09116"
pdf_path: "assets/papers/NaturalSpeech2.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, tts]
created: 2026-05-22
---

# NaturalSpeech 2: Latent Diffusion for Zero-Shot TTS

## 📌 一句话

微软提出的 zero-shot TTS/SVS 系统——在 **latent diffusion** 框架上做语音合成，用 neural audio codec（[[RVQ]]）提取连续 latent + diffusion model 生成，支持 zero-shot 多说话人和歌声合成。

## 🛠 核心方法

**输入 → 输出**: phoneme + speech prompt → latent → waveform

**架构组件**（按数据流顺序）:
1. **Neural Audio Codec**: encoder + RVQ + decoder，提取连续 latent representation
2. **Phoneme Encoder + Duration/Pitch Predictor**: 文本条件 + 韵律预测
3. **Latent Diffusion Model**: WaveNet-based diffusion，在 codec latent 空间生成
4. **Speech Prompting**: in-context learning 机制，用 reference speech 做 zero-shot 条件

**关键创新**: 首次将 **latent diffusion** 引入语音合成——不在波形空间做 diffusion（太慢），而在 codec latent 空间做（更高效），同时支持语音和歌声的 zero-shot 合成。

## 🖼 架构图

![Figure 1: NaturalSpeech 2 — audio codec + latent diffusion model 总览](https://ar5iv.labs.arxiv.org/html/2304.09116/assets/x1.png)

## 📊 关键结果 / 评测

- Zero-shot TTS: speaker similarity 优于 VALL-E
- 支持歌声合成（SVS）
- 训练数据: 44K 小时

## 💡 借鉴意义（一句话）

做 TTS / 扩散模型的人关注——NaturalSpeech 2 证明了 latent diffusion 在语音合成中的有效性，是 [[NaturalSpeech3]] 和众多 diffusion TTS 的直接前身。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2304.09116
- PDF: [[assets/papers/NaturalSpeech2.pdf|本地 PDF]]
- 源目录: `TTS-LLM/NS2.pdf`
