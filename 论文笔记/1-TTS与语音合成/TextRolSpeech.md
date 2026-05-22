---
title: "TextrolSpeech: A Text Style Control Speech Corpus with Codec Language Text-to-Speech Models"
method_name: "TextRolSpeech"
authors: [Shengpeng Ji, Jialong Zuo, Minghui Fang, Ziyue Jiang, Feiyang Chen]
year: 2023
venue: arXiv
arxiv_id: "2308.14430"
pdf_path: "assets/papers/TextRolSpeech.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, tts, dataset]
created: 2026-05-22
---

# TextRolSpeech: Text Style Control Speech Corpus

## 📌 一句话

浙大/华为提出的**风格可控语音数据集**——构建了大规模文本风格标注语音数据集 TextRolSpeech，并基于此训练 codec language model 实现文本描述驱动的风格化 TTS。

## 🛠 核心方法

**输入 → 输出**: text + style description → styled speech

**架构组件**:
1. **TextRolSpeech Dataset**: 大规模风格标注语音数据集（风格用自然语言描述）
2. **Style Encoder**: 编码风格描述文本
3. **Codec Language Model**: 条件于风格的 speech token 生成

**关键创新**: 构建了**文本风格描述 + 语音**的配对数据集——让 TTS 可以通过自然语言描述来控制说话风格，而非传统的 style embedding。

## 📊 关键结果 / 评测

- 提供大规模风格标注数据集
- 文本描述驱动的风格 TTS

## 💡 借鉴意义（一句话）

做风格可控 TTS / 数据集的人参考——TextRolSpeech 提供了"文本风格描述→语音"的数据和方法。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2308.14430
- PDF: [[assets/papers/TextRolSpeech.pdf|本地 PDF]]
- 源目录: `TTS-LLM/text-role-speech.pdf`
