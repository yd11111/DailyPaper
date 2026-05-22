---
title: "CosyVoice 2: Scalable Streaming Speech Synthesis with Large Language Models"
method_name: "CosyVoice2"
authors: [Zhihao Du, Yuxuan Wang, Qian Chen, Xian Shi, Xiang Lv]
year: 2024
venue: arXiv
arxiv_id: "2412.10117"
pdf_path: "assets/papers/CosyVoice2.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, tts]
created: 2026-05-22
---

# CosyVoice 2: Streaming TTS with LLM

## 📌 一句话

[[CosyVoice]] 的升级版——核心改进是**统一 streaming/non-streaming 架构**（同一个模型同时支持流式和非流式推理）+ chunk-aware causal flow matching，实现低延迟流式合成而不损失非流式质量。

## 🛠 核心方法

**输入 → 输出**: text + speech prompt → semantic tokens → mel (streaming) → waveform

**架构组件**（按数据流顺序）:
1. **Supervised Speech Tokenizer**: 继承 CosyVoice 1 的有监督 tokenizer
2. **Unified Text-Speech LM**: 统一的 LLM 支持 streaming + non-streaming
3. **Chunk-aware Causal Flow Matching**: 按 chunk 做因果 flow matching，支持流式推理
4. **Vocoder**: mel → waveform

**关键创新**: **统一 streaming/non-streaming**——通过 chunk-aware 设计让同一个模型在训练时同时学习两种模式，推理时按需切换，不需要维护两套模型。

## 🖼 架构图

![Figure 1: CosyVoice 2 总览——supervised tokenizer + unified LM + causal flow matching](https://ar5iv.labs.arxiv.org/html/2412.10117/assets/x1.png)

## 📊 关键结果 / 评测

- LibriSpeech test-clean: WER 2.47%, NMOS 3.96, speaker similarity 0.745（均优于 human recording）
- Seed-TTS-eval test-zh: CER 1.45%, speaker similarity 0.806
- Seed-TTS-eval test-en: WER 2.57%
- 流式模式（CosyVoice 2-S）质量几乎无损：test-zh CER 保持 1.45%

## 💡 借鉴意义（一句话）

做流式 TTS 的人关注——CosyVoice 2 的统一 streaming/non-streaming 设计是工程上非常实用的方案。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2412.10117
- PDF: [[assets/papers/CosyVoice2.pdf|本地 PDF]]
- 源目录: `TTS-LLM/cosy2.pdf`
