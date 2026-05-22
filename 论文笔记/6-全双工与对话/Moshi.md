---
title: "Moshi: a speech-text foundation model for real-time dialogue"
method_name: "Moshi"
authors: [Alexandre Défossez, Laurent Mazaré, Manu Ber, Neil Zeghidour]
year: 2024
venue: arXiv
arxiv_id: "2410.00037"
pdf_path: "assets/papers/Moshi.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, full-duplex]
created: 2026-05-22
---

# Moshi: Speech-Text Foundation Model for Real-Time Dialogue

## 📌 一句话

Kyutai 提出的**全双工语音对话模型**——同时建模用户和系统两路音频流 + 内部文本思维链（Inner Monologue），实现真正的实时对话（160ms 理论延迟），是首个开源的端到端 speech-to-speech 对话系统。

## 🛠 核心方法

**输入 → 输出**: user speech stream ↔ system speech stream (full-duplex)

**架构组件**（按架构层次）:
1. **Mimi Codec**: 改进的 [[SoundStream]] codec，分离 semantic token 和 acoustic token（distillation from [[WavLM]]）
2. **Helium LM**: 7B text LM backbone（基于 Llama 架构）
3. **Dual-stream Modeling**: 同时自回归建模 user stream + system stream
4. **Inner Monologue**: 系统在生成语音的同时生成内部文本 token，引导语义推理

**关键创新**: 用 **Inner Monologue** 巧妙地在语音生成中注入文本推理能力——模型先"想"出文本再说出语音，兼顾了语音的实时性和文本 LLM 的推理能力。

## 🖼 架构图

![Figure 1: Moshi — 双流建模 + Inner Monologue 架构](https://ar5iv.labs.arxiv.org/html/2410.00037/assets/figures/overview_moshi_v2.png)

## 📊 关键结果 / 评测

- 端到端延迟: 160ms 理论 / 200ms 实测（低于自然对话平均 230ms）
- Mimi codec: MUSHRA 81.0 @ 1.1kbps（优于 SpeechTokenizer 4.0kbps 的 74.3）
- Inner Monologue: transcript NLL 3.65→2.77，生成文本长度 602→1920 字符
- Helium LM: MMLU 54.3（优于同规模 Llama 2 的 45.3）

## 💡 借鉴意义（一句话）

做全双工对话 / Speech LLM 的人**必读**——Moshi 的 Inner Monologue + 双流架构是目前开源全双工方案的标杆。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2410.00037
- PDF: [[assets/papers/Moshi.pdf|本地 PDF]]
- 源目录: `TTS-LLM/moshi.pdf`
