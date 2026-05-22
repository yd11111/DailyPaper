---
title: "Step-Audio-EditX Technical Report"
method_name: "StepAudio-EditX"
authors: [Chao Yan, Boyong Wu, Peng Yang, Pengfei Tan, Guoqiang Hu]
year: 2025
venue: arXiv
arxiv_id: "2511.03601"
pdf_path: "assets/papers/StepAudio-EditX.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, speech-llm]
created: 2026-05-22
---

# Step-Audio-EditX: Expressive & Iterative Audio Editing

## 📌 一句话

[[StepAudio]] 的语音编辑扩展——首个开源的 **LLM-based 语音编辑模型**，支持情感/语速/停顿/方言的迭代编辑（输入语音 + 文本指令 → 编辑后的语音），保持说话人身份不变。

## 🛠 核心方法

**输入 → 输出**: source speech + edit instruction → edited speech

**架构组件**:
1. **Speech Encoder**: 编码源语音
2. **LLM**: 理解编辑指令 + 生成编辑后的 speech token
3. **Speech Decoder**: 合成编辑后的语音

**关键创新**: 首个支持**迭代编辑**的语音编辑系统——可以多轮编辑同一段语音（先改情感 → 再改语速 → 再改停顿），每轮保持说话人一致。

## 📊 关键结果 / 评测

- 开源（模型 + 代码）
- 支持情感/语速/停顿/方言多维度编辑

## 💡 借鉴意义（一句话）

做语音编辑 / Speech LLM 的人关注——StepAudio-EditX 展示了 LLM-based 语音编辑的可行性。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2511.03601
- PDF: [[assets/papers/StepAudio-EditX.pdf|本地 PDF]]
- 源目录: `TTS-LLM/StepAudio-EditX.pdf`
