---
title: "FireRedTTS: A Foundation Text-To-Speech Framework for Industry-Level Generative Speech Applications"
method_name: "FireRedTTS"
authors: [FireRed Team, Xiaohongshu]
year: 2024
venue: arXiv
arxiv_id: "2409.03283"
pdf_path: "assets/papers/FireRedTTS.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, tts]
created: 2026-05-22
---

# FireRedTTS: Foundation TTS for Industry-Level Applications

## 📌 一句话

小红书推出的**工业级 TTS 框架**——三部分组成：数据处理 pipeline + TTS language model（text → semantic token）+ token-to-waveform generator（flow matching + mel codec），强调工业落地的数据和工程最佳实践。

## 🛠 核心方法

**输入 → 输出**: text + speaker prompt → semantic tokens → mel → waveform

**架构组件**（按数据流顺序）:
1. **Data Pipeline**: 语音增强 → 切分 → 说话人聚类 → ASR 转写 → 质量过滤
2. **TTS Language Model**: text token → semantic token（AR 生成）
3. **Flow Matching Mel Decoder**: semantic → mel（flow matching 模型）
4. **Streamable Decoder**: multi-stream LM + Mel Codec（支持流式推理）

**关键创新**: 系统性地分享了**工业级 TTS 的数据处理经验**——从数据质量的角度优化 TTS 效果，同时提出 streamable decoder 支持实时推理。

## 🖼 架构图

![Figure 3: FireRedTTS 系统总览——TTS LM + flow matching decoder](https://ar5iv.labs.arxiv.org/html/2409.03283/assets/image/fireredtts.png)

## 📊 关键结果 / 评测

- 一致性 CoMOS: 4.32（CosyVoice 4.15, GT 4.53）
- 中文发音错误率: 2.09%（CosyVoice 5.68%）
- Few-shot PUGC voice cloning: MOS 4.65, SIM 78.92%
- 情感控制: instruction tuning 后分类准确率 50%→97%

## 💡 借鉴意义（一句话）

做 TTS 工业落地的人参考——FireRedTTS 的数据 pipeline 和 streamable decoder 设计有较强的工程参考价值。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2409.03283
- PDF: [[assets/papers/FireRedTTS.pdf|本地 PDF]]
- 源目录: `TTS-LLM/fireredTTS.pdf`
