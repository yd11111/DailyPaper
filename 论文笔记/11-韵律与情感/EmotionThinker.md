---
title: "EmotionThinker: Prosody-Aware Reinforcement Learning for Explainable Speech Emotion Reasoning"
method_name: "EmotionThinker"
authors: []
year: 2026
venue: ICLR 2026 submission
arxiv_id: null
pdf_path: "assets/papers/EmotionThinker.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, emotion, speech-llm]
created: 2026-05-22
---

# EmotionThinker: Prosody-Aware RL for Speech Emotion Reasoning

## 📌 一句话

提出用 **RL + prosody-aware 特征** 做可解释的语音情感推理——让 Speech LLM 不仅输出情感标签，还生成推理过程（为什么判断为这个情感），通过 RL 训练提升推理质量。

## 🛠 核心方法

**输入 → 输出**: speech → prosody features + reasoning chain → emotion label + explanation

**架构组件**:
1. **Speech Encoder**: 提取语音特征（含韵律）
2. **Prosody-Aware Module**: 显式提取韵律信息（pitch / energy / tempo）
3. **Reasoning LLM**: 生成推理链解释情感判断
4. **RL Training**: 通过 reward model 优化推理质量

**关键创新**: 将 **chain-of-thought reasoning** 引入语音情感识别——不只给标签，还解释"为什么"，RL 确保推理过程准确而非编造。

## 📊 关键结果 / 评测

- ICLR 2026 submission（anonymous）
- 情感分类准确率和推理可解释性双提升

## 💡 借鉴意义（一句话）

做语音情感 / Speech LLM 的人了解——EmotionThinker 探索了语音领域的 reasoning + RL 结合。

## 🔗 链接

- PDF: [[assets/papers/EmotionThinker.pdf|本地 PDF]]
- 源目录: `TTS-LLM/EMOTIONTHINKER.pdf`
