---
title: "NaturalSpeech 3: Zero-Shot Speech Synthesis with a Factorized Codec and Diffusion Models"
method_name: "NaturalSpeech3"
authors: [Zeqian Ju, Yuancheng Wang, Kai Shen, Xu Tan, Detai Xin]
year: 2024
venue: ICML 2024
arxiv_id: "2403.03100"
pdf_path: "assets/papers/NaturalSpeech3.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, tts]
created: 2026-05-22
---

# NaturalSpeech 3: Factorized Codec + Factorized Diffusion

## 📌 一句话

微软提出的第三代 zero-shot TTS——核心创新是 **FACodec**（将语音属性分解为 content / prosody / timbre / acoustic detail 四个独立 VQ 空间）+ **factorized diffusion model**（每个属性用独立 diffusion 模块生成），实现更精细的属性控制。

## 🛠 核心方法

**输入 → 输出**: phoneme + speech prompt → factorized latent → waveform

**架构组件**（按数据流顺序）:
1. **FACodec (Factorized Audio Codec)**: 将语音编码为 4 个解耦的 VQ 子空间（content / prosody / timbre / acoustic detail）
2. **Phoneme Encoder + Duration Diffusion**: 文本编码 + 可微分时长生成
3. **Prosody Diffusion**: 生成韵律 latent
4. **Content Diffusion**: 生成内容 latent
5. **Detail Diffusion**: 生成声学细节 latent

**关键创新**: 通过**属性分解**（factorization）让每个 diffusion 子模块只需要建模一个维度的变化，大幅降低建模复杂度，生成质量和可控性同时提升。

## 🖼 架构图

![Figure 2: FACodec — 语音属性分解为 content/prosody/timbre/detail 四路 VQ](https://ar5iv.labs.arxiv.org/html/2403.03100/assets/x2.png)

![Figure 3: Factorized Diffusion Model — 按属性分解的多路 diffusion](https://ar5iv.labs.arxiv.org/html/2403.03100/assets/x3.png)

## 📊 关键结果 / 评测

- LibriSpeech test-clean: WER 1.81%, SIM 0.67
- 显著优于 VALL-E / NaturalSpeech 2
- 训练数据: 200K 小时

## 💡 借鉴意义（一句话）

做 TTS / Audio Codec 的人**必读**——FACodec 的属性分解思路对 codec 设计有重要启发，factorized diffusion 是 diffusion TTS 的一个重要方向。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2403.03100
- PDF: [[assets/papers/NaturalSpeech3.pdf|本地 PDF]]
- 源目录: `TTS-LLM/ns3.pdf`
