---
title: "Robust Speech Recognition via Large-Scale Weak Supervision"
method_name: "Whisper"
authors: [Alec Radford, Jong Wook Kim, Tao Xu, Greg Brockman, Christine McLeavey, Ilya Sutskever]
year: 2022
venue: ICML 2023
arxiv_id: null
pdf_path: "assets/papers/Whisper.pdf"
library_source: "高德文献库"
source_topic: "SSL"
tags: [classic, asr]
created: 2026-05-22
---

# Whisper: Large-Scale Weakly-Supervised ASR

## 📌 一句话

OpenAI 用 680,000 小时互联网多语言弱监督音频训练的 encoder-decoder ASR 模型，zero-shot 泛化到几乎所有 benchmark 上接近甚至超过 fully-supervised 模型，是当前语音处理领域的通用基座之一。

## 🛠 核心方法

**输入 → 输出**: 30s log-mel spectrogram → text tokens（多任务: transcription / translation / language-id / timestamps）

**架构组件**（按数据流顺序）:
1. **Audio Encoder (Transformer)**: 80-channel log-mel → 2 层 Conv1D 下采样 → Transformer blocks
2. **Text Decoder (Transformer)**: 自回归解码，输入为 special tokens 序列（`<|lang|> <|task|> <|notimestamps|>`）控制任务
3. **Multitask Training Format**: 同一个模型同时学 ASR / 翻译 / 语种识别 / 时间戳对齐，靠 prompt token 切换
4. **数据规模**: 680K 小时互联网音频（弱标签），覆盖 96 种语言

**关键创新**: 不做 self-supervised pre-training（与 wav2vec / HuBERT 路线相反），直接用**海量弱监督数据 + 简单 encoder-decoder** 暴力 scale，证明数据量能弥补标签噪声。

## 🖼 架构图

![Whisper spectrogram 示意（pdf_tools fallback，非架构图）](assets/papers/figs/Whisper_fig1.png)

## 📊 关键结果 / 评测

- LibriSpeech test-clean: WER 2.5%（zero-shot，无 LM）
- Fleurs 多语言: 覆盖 96 语种，平均 WER ~30%（zero-shot）
- 鲁棒性: 在噪声/口音/领域迁移场景下显著优于 supervised 基线

## 💡 借鉴意义（一句话）

做 ASR / Speech LLM / 全双工的人**必读**——Whisper encoder 是目前最常用的语音特征提取器之一，几乎所有 Speech LLM（Qwen-Audio、SALMONN 等）都用它做前端。

## 🔗 链接

- PDF: [[assets/papers/Whisper.pdf|本地 PDF]]
- 源目录: `SSL/whisper.pdf`
