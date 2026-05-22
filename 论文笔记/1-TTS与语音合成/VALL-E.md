---
title: "Neural Codec Language Models are Zero-Shot Text to Speech Synthesizers"
method_name: "VALL-E"
authors: [Chengyi Wang, Sanyuan Chen, Yu Wu, Ziqiang Zhang, Long Zhou]
year: 2023
venue: arXiv
arxiv_id: "2301.02111"
pdf_path: "assets/papers/VALL-E.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, tts, speech-llm]
created: 2026-05-22
---

# VALL-E: Neural Codec Language Models are Zero-Shot TTS

## 📌 一句话

微软提出将 TTS 重新定义为**条件语言建模**任务——用 [[EnCodec]] 把语音离散化为 token，再用 AR + NAR transformer 预测这些 token，仅需 3 秒 prompt 即可 zero-shot 合成任意说话人语音。**开创了 codec LM TTS 范式**。

## 🛠 核心方法

**输入 → 输出**: text + 3s speech prompt → EnCodec tokens → synthesized speech

**架构组件**（按数据流顺序）:
1. **EnCodec Tokenizer**: 将语音离散化为 8 层 RVQ code
2. **AR Model**: 自回归预测第 1 层 codec token（粗粒度，决定韵律和内容）
3. **NAR Model**: 非自回归并行预测第 2-8 层 codec token（细粒度，补充音质细节）
4. **EnCodec Decoder**: 从 8 层 token 重建波形

**关键创新**: 首次证明**大规模语音数据 + codec language model**可以实现高质量 zero-shot TTS，打破了传统 TTS 对 speaker embedding 的依赖，开启了后续 VALL-E 2 / VALL-E X / [[MaskGCT]] / [[SeedTTS]] 等一系列工作。

## 🖼 架构图

![Figure 1: VALL-E — 将 TTS 视为 conditional codec language model](https://ar5iv.labs.arxiv.org/html/2301.02111/assets/prompt.jpg)

## 📊 关键结果 / 评测

- LibriSpeech: speaker similarity SMOS 3.8（显著优于 YourTTS 等 baseline）
- Zero-shot TTS: 仅需 3 秒 enrollment，无需微调
- 训练数据: 60K 小时 LibriLight

## 💡 借鉴意义（一句话）

做 TTS / Speech LLM 的人**必读**——VALL-E 是 codec language model TTS 的开山之作，后续几乎所有 LLM-based TTS 都沿用或改进了这一范式。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2301.02111
- PDF: [[assets/papers/VALL-E.pdf|本地 PDF]]
- 源目录: `TTS-LLM/valle.pdf`
