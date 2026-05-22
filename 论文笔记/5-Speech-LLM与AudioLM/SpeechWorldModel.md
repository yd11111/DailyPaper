---
title: "Speech World Model: Causal State-Action Planning with Explicit Reasoning for Speech"
method_name: "SpeechWorldModel"
authors: []
year: 2026
venue: ICLR 2026 submission
arxiv_id: null
pdf_path: "assets/papers/SpeechWorldModel.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, speech-llm]
created: 2026-05-22
---

# Speech World Model: Causal State-Action Planning for Speech

## 📌 一句话

提出将 **world model** 思想引入语音理解——把语音理解建模为 causal state-action planning + explicit reasoning，让模型先推理再给出答案，提升复杂语音任务的推理能力。

## 🛠 核心方法

**输入 → 输出**: speech → explicit reasoning chain → answer

**架构组件**（按推理流程）:
1. **Speech Encoder**: 语音输入编码
2. **World Model**: 因果状态-动作建模
3. **Explicit Reasoning**: 逐步推理链
4. **Answer Generation**: 基于推理结果生成答案

**关键创新**: 将 world model 的因果推理思路引入语音领域，让语音理解不再是简单的端到端映射，而是包含显式推理过程。

## 📊 关键结果 / 评测

- ICLR 2026 submission（anonymous）
- 在语音推理任务上优于直接端到端方法

## 💡 借鉴意义（一句话）

做 Speech LLM / 语音推理的人了解——Speech World Model 探索了语音领域的 reasoning 能力，方向前沿。

## 🔗 链接

- PDF: [[assets/papers/SpeechWorldModel.pdf|本地 PDF]]
- 源目录: `TTS-LLM/Speech World Model.pdf`
