---
title: "A Layer-wise Analysis of Mandarin and English Suprasegmentals in SSL Speech Models"
method_name: "SSL-Suprasegmental"
authors: [Antón de la Fuente, Dan Jurafsky]
year: 2024
venue: Interspeech 2024
arxiv_id: "2408.13678"
pdf_path: "assets/papers/SSL-Suprasegmental.pdf"
library_source: "高德文献库"
source_topic: "SSL"
tags: [classic, ssl]
created: 2026-05-22
---

# SSL-Suprasegmental: Layer-wise Analysis of Prosody in SSL Models

## 📌 一句话

Stanford 对 wav2vec 2.0 / [[HuBERT]] / [[WavLM]] 各层编码**超音段信息**（重音、声调、F0）的能力做系统 probing 分析，发现不同模型和层级对韵律信息的编码方式显著不同。

## 🛠 核心方法

**输入 → 输出**: 分析论文，无独立模型

**架构组件**（按分析维度）:
1. **Probing Tasks**: English word stress / English accent / Mandarin tone / F0 regression
2. **Models**: wav2vec 2.0 / HuBERT / WavLM（英语 + 中文单语版本）
3. **Layer-wise Analysis**: 逐层（0-12 层）测 probing 准确率，定位各层的韵律编码能力
4. **Pre-trained vs Fine-tuned**: 对比预训练和 ASR 微调后的韵律表示变化

**关键创新**: 首次跨模型 + 跨语言（英中）地分析 SSL 模型的超音段信息编码，发现 WavLM 对韵律最敏感、HuBERT 次之、wav2vec 2.0 最弱。

## 📊 关键结果 / 评测

- WavLM 在韵律 probing 上一致优于 HuBERT 和 wav2vec 2.0
- 中间层（5-8 层）韵律信息最丰富
- ASR 微调后韵律表示退化（task specialization 代价）

## 💡 借鉴意义（一句话）

做 TTS prosody / SSL 表示学习的人参考——选 WavLM 做韵律提取器优于 HuBERT；从中间层取特征优于最后一层。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2408.13678
- PDF: [[assets/papers/SSL-Suprasegmental.pdf|本地 PDF]]
- 源目录: `SSL/SSL-anays.pdf`
