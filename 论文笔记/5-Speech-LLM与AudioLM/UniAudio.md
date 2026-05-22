---
title: "UniAudio: An Audio Foundation Model Toward Universal Audio Generation"
method_name: "UniAudio"
authors: [Dongchao Yang, Jinchuan Tian, Xu Tan, Rongjie Huang, Songxiang Liu]
year: 2023
venue: arXiv
arxiv_id: "2310.00704"
pdf_path: "assets/papers/UniAudio.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, speech-llm]
created: 2026-05-22
---

# UniAudio: Universal Audio Generation Foundation Model

## 📌 一句话

港中文/USTC/微软合作的**统一音频生成模型**——将 TTS / 音乐 / 音效 / 语音增强等 11 种音频任务统一为"token-to-token"框架，用 multi-scale transformer 处理 [[EnCodec]] 的多层 RVQ token。

## 🛠 核心方法

**输入 → 输出**: various conditions (text/phoneme/audio) → audio tokens → waveform

**架构组件**（按数据流顺序）:
1. **Tokenizers**: 各任务输入统一 tokenize（phoneme / semantic / EnCodec token）
2. **Multi-scale Transformer**: 全局 transformer + 局部 transformer 处理多层 RVQ
3. **EnCodec Decoder**: audio token → waveform

**关键创新**: 用**统一的 next-token prediction 框架**处理 11 种音频生成任务（TTS / VC / SE / singing / music / ...），multi-scale transformer 高效处理 RVQ 多层 token。

## 🖼 架构图

![Figure 1: UniAudio 总览——统一框架 + multi-scale transformer](https://ar5iv.labs.arxiv.org/html/2310.00704/assets/x1.png)

## 📊 关键结果 / 评测

- TTS: SIM 0.71, WER 2.0, MOS 3.81（优于 NaturalSpeech 2 的 SIM 0.62）
- Text-to-Sound: FAD 3.12（AudioLDM 4.93）
- Text-to-Music: FAD 3.65（MusicGen 4.52）
- 多任务联合训练一致提升：TTS SIM 0.64→0.71，Sound FAD 3.84→3.12

## 💡 借鉴意义（一句话）

做 Audio Foundation Model 的人关注——UniAudio 展示了统一音频生成框架的可行性，multi-scale transformer 对 RVQ token 的处理方式值得借鉴。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2310.00704
- PDF: [[assets/papers/UniAudio.pdf|本地 PDF]]
- 源目录: `TTS-LLM/UniAudio.pdf`
