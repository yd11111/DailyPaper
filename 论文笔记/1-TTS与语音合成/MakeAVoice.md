---
title: "Make-A-Voice: Unified Voice Synthesis With Discrete Representation"
method_name: "MakeAVoice"
authors: [Rongjie Huang, Chunlei Zhang, Yongqi Wang, Dongchao Yang, Luping Liu]
year: 2023
venue: arXiv
arxiv_id: "2305.19269"
pdf_path: "assets/papers/MakeAVoice.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, tts]
created: 2026-05-22
---

# Make-A-Voice: Unified Voice Synthesis

## 📌 一句话

浙大/腾讯提出的**统一语音合成框架**——用离散 token 统一 TTS / VC / 歌声合成三个任务，三阶段 coarse-to-fine 架构（semantic → acoustic → waveform）。

## 🛠 核心方法

**输入 → 输出**: text/speech → semantic tokens → acoustic tokens → waveform

**架构组件**（按数据流顺序）:
1. **Semantic Stage**: text/speech → HuBERT-based semantic token
2. **Acoustic Stage**: semantic → fine-grained acoustic token
3. **Generation Stage**: unit-based vocoder → waveform

**关键创新**: 用同一套**离散 token + coarse-to-fine 框架**统一了 TTS / VC / SVS 三个任务，不需要为每个任务单独设计模型。

## 🖼 架构图

![Figure 1: Make-A-Voice — 三阶段 coarse-to-fine 统一框架](https://ar5iv.labs.arxiv.org/html/2305.19269/assets/x1.png)

## 📊 关键结果 / 评测

- TTS: MOS 4.04, speaker cosine 0.77, CER 0.068（优于 YourTTS / GenerSpeech）
- VC: MOS 4.07, speaker cosine 0.80
- SVS: MOS 3.99, FFE 0.05（pitch 精度最优）

## 💡 借鉴意义（一句话）

做统一语音合成的人参考——Make-A-Voice 展示了离散 token 框架统一多个语音任务的可行性。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2305.19269
- PDF: [[assets/papers/MakeAVoice.pdf|本地 PDF]]
- 源目录: `TTS-LLM/MakeAVoice.pdf`
