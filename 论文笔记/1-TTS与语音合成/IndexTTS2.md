---
title: "IndexTTS2: A Breakthrough in Emotionally Expressive and Duration-Controlled Auto-Regressive Zero-Shot Text-to-Speech"
method_name: "IndexTTS2"
authors: [Siyi Zhou, Yiquan Zhou, Yi He, Xun Zhou, Jinchao Wang, Wei Deng]
year: 2025
venue: arXiv
arxiv_id: "2506.21619"
pdf_path: "assets/papers/IndexTTS2.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, tts]
created: 2026-05-22
---

# IndexTTS2: Emotionally Expressive & Duration-Controlled TTS

## 📌 一句话

B站推出的 zero-shot TTS 升级版——在 AR 框架中加入**情感表达控制**和**时长控制**，解决了 AR TTS 难以精确控制韵律时长和情感表达的问题。

## 🛠 核心方法

**输入 → 输出**: text + reference audio + emotion/duration control → speech

**架构组件**:
1. **AR Model**: 自回归生成 speech token
2. **Emotion Control**: 情感条件注入
3. **Duration Control**: 细粒度时长控制
4. **Decoder**: token → waveform

**关键创新**: 在 AR zero-shot TTS 中实现**精确的情感和时长控制**——不只是 clone 说话人音色，还能指定情感类型和语速。

## 📊 关键结果 / 评测

- 情感表达：多种情感类型控制
- 时长控制：精确到字级别
- 开源可用

## 💡 借鉴意义（一句话）

做可控 TTS 的人了解——IndexTTS2 在 AR 框架中同时实现情感和时长控制，工程设计值得参考。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2506.21619
- PDF: [[assets/papers/IndexTTS2.pdf|本地 PDF]]
- 源目录: `TTS-LLM/indexTTS2.pdf`
