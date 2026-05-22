---
title: "Analysing the Language of Neural Audio Codecs"
method_name: "AnalysingCodecLanguage"
authors: [Joonyong Park, Shinnosuke Takamichi, David M. Chan, Shunsuke Kando]
year: 2025
venue: arXiv
arxiv_id: "2509.01390"
pdf_path: "assets/papers/AnalysingCodecLanguage.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, eval, codec]
created: 2026-05-22
---

# Analysing the Language of Neural Audio Codecs

## 📌 一句话

对神经音频 codec 产出的 **token 序列进行语言学分析**——把 codec token 当作一种"语言"，用 NLP 方法（zipf's law / perplexity / 互信息等）分析其统计和语言学特性，帮助理解为什么某些 codec 对 LM 更友好。

## 🛠 核心方法

**输入 → 输出**: codec token sequences → statistical/linguistic analysis

**架构组件**:
1. **Codec Token Extraction**: 从多个 codec（[[EnCodec]] / [[SoundStream]] / [[DAC]] 等）提取 token 序列
2. **Statistical Analysis**: Zipf's law / entropy / vocabulary usage
3. **Linguistic Analysis**: token 间互信息 / 序列可预测性 / perplexity

**关键创新**: 首次系统性地将 **NLP 语言分析工具**应用于 audio codec token，揭示了不同 codec token 的"语言特性"差异，解释了为什么某些 codec 对 LM 建模更友好。

## 📊 关键结果 / 评测

- 不同 codec 的 token 统计特性差异显著
- "更语言化"的 token（更接近 Zipf 分布）对 LM 更友好

## 💡 借鉴意义（一句话）

做 Audio Codec / Speech LLM 的人参考——理解 codec token 的"语言特性"有助于设计对 LM 更友好的 tokenizer。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2509.01390
- PDF: [[assets/papers/AnalysingCodecLanguage.pdf|本地 PDF]]
- 源目录: `TTS-LLM/2509.01390v1.pdf`
