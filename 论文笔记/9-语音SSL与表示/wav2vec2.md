---
title: "wav2vec 2.0: A Framework for Self-Supervised Learning of Speech Representations"
method_name: "wav2vec2"
authors: [Alexei Baevski, Henry Zhou, Abdelrahman Mohamed, Michael Auli]
year: 2020
venue: NeurIPS 2020
arxiv_id: "2006.11477"
pdf_path: "assets/papers/wav2vec2.pdf"
library_source: "高德文献库"
source_topic: "SSL"
tags: [classic, ssl]
created: 2026-05-22
---

# wav2vec 2.0: Self-Supervised Speech Representation

## 📌 一句话

Meta AI 奠基性的语音自监督学习框架——对比学习 + 量化 + masked prediction，用 53K 小时无标注语音预训练后仅需 10 分钟标注数据就能做到 competitive ASR，开启了语音 SSL 革命。

## 🛠 核心方法

**输入 → 输出**: raw waveform → contextualized speech representations

**架构组件**（按数据流顺序）:
1. **CNN Feature Encoder**: 7 层 CNN 把波形编码为帧级特征
2. **Quantization Module**: Gumbel-softmax 量化 CNN 输出为离散 token
3. **Transformer Context Network**: 对 masked 位置的 CNN 特征做 contextualization
4. **Contrastive Loss**: 让 Transformer 输出对齐正确的量化 token（区分 negatives）

**关键创新**: 首次证明了语音领域"大规模无标注预训练 + 少量标注 fine-tune"的范式可行性——**10 分钟标注 + 53K 小时无标注 = 竞争性 ASR**，改变了整个领域。

## 🖼 架构图

![Figure 1: wav2vec 2.0 framework — feature encoder + quantization + context network with contrastive learning](https://ar5iv.labs.arxiv.org/html/2006.11477/assets/x1.png)

## 📊 关键结果 / 评测

- LibriSpeech test-clean: WER 1.8%（Large + LM）
- 10 min labeled: WER 4.8%（test-other），证明极低标注量 ASR 可行
- 100 小时 labeled: 接近全量 960h supervised baseline

## 💡 借鉴意义（一句话）

语音 AI 从业者**必读**——wav2vec 2.0 开创了语音 SSL 范式，[[HuBERT]] / [[WavLM]] / [[w2v-BERT]] 都是它的后续迭代。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2006.11477
- PDF: [[assets/papers/wav2vec2.pdf|本地 PDF]]
- 源目录: `SSL/wav2vec2.0.pdf`
