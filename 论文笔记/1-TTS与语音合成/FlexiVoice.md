---
title: "FlexiVoice: Enabling Flexible Style Control in Zero-Shot TTS with Natural Language Instructions"
method_name: "FlexiVoice"
authors: []
year: 2026
venue: ICLR 2026 submission
arxiv_id: null
pdf_path: "assets/papers/FlexiVoice.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, tts]
created: 2026-05-22
---

# FlexiVoice: Flexible Style Control with NL Instructions

## 📌 一句话

提出用**自然语言指令**控制 zero-shot TTS 的风格——不需要参考音频来传递风格，而是通过文字描述（如"用低沉柔和的声音说"）控制语速/情感/说话风格。

## 🛠 核心方法

**输入 → 输出**: text + natural language style instruction → styled speech

**架构组件**:
1. **Style Instruction Encoder**: 编码自然语言风格描述
2. **Zero-shot TTS Model**: 条件语音合成
3. **Style Conditioning**: 将 NL 风格指令注入合成过程

**关键创新**: 用**自然语言**替代传统的参考音频 / style embedding 做风格控制——更灵活，用户不需要准备参考音频。

## 📊 关键结果 / 评测

- ICLR 2026 submission（anonymous）
- 自然语言风格控制 + zero-shot voice cloning

## 💡 借鉴意义（一句话）

做可控 TTS 的人了解——FlexiVoice 的 NL-based 风格控制是 TTS 可控性的一个新方向。

## 🔗 链接

- PDF: [[assets/papers/FlexiVoice.pdf|本地 PDF]]
- 源目录: `TTS-LLM/FLEXIVOICE.pdf`
