---
title: "W2V-BERT: Combining Contrastive Learning and Masked Language Modeling for Self-Supervised Speech Pre-Training"
method_name: "w2v-BERT"
authors: [Yu-An Chung, Yu Zhang, Wei Han, Chung-Cheng Chiu, James Qin]
year: 2021
venue: ASRU 2021
arxiv_id: "2108.06209"
pdf_path: "assets/papers/w2v-BERT.pdf"
library_source: "高德文献库"
source_topic: "SSL"
tags: [classic, ssl]
created: 2026-05-22
---

# w2v-BERT: Contrastive + MLM for Speech SSL

## 📌 一句话

Google 提出将 [[wav2vec 2.0]] 的**对比学习**和 BERT 的 **Masked Language Modeling (MLM)** 结合到一个模型中——前 N 层做 contrastive（学离散单元），后 M 层做 MLM（学上下文表示），两阶段端到端联合训练。

## 🛠 核心方法

**输入 → 输出**: raw waveform → contextualized speech representations

**架构组件**（按数据流顺序）:
1. **Feature Encoder**: CNN 提取帧级特征
2. **Contrastive Module (前 N 层 Conformer)**: 对比学习生成离散 token（类似 wav2vec 2.0 的量化模块）
3. **MLM Module (后 M 层 Conformer)**: 以 contrastive 产出的离散 token 为目标做 masked prediction
4. **端到端训练**: contrastive + MLM 联合优化，无需两阶段

**关键创新**: 用一个模型统一了 wav2vec 2.0（contrastive）和 HuBERT（MLM）两条路线——contrastive 做 tokenizer，MLM 做 contextualizer，避免了 HuBERT 的 k-means 离线聚类步骤。

## 🖼 架构图

![Figure 1: w2v-BERT framework — contrastive module + MLM module](https://ar5iv.labs.arxiv.org/html/2108.06209/assets/x1.png)

## 📊 关键结果 / 评测

- LibriSpeech test-clean: WER 1.4%（60K 小时预训练 + fine-tune）
- 优于 HuBERT-Large / wav2vec 2.0-Large

## 💡 借鉴意义（一句话）

做语音 SSL 的人了解即可——w2v-BERT 2.0 是 [[SeamlessM4T]] 的语音编码器，但 v1 本身已被 WavLM / HuBERT 系列追赶。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2108.06209
- PDF: [[assets/papers/w2v-BERT.pdf|本地 PDF]]
- 源目录: `SSL/w2vBert.pdf`
