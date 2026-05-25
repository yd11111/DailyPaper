---
title: "MambaVoiceCloning: Efficient and Expressive Text-to-Speech via State-Space Modeling and Diffusion Control"
method_name: "MambaVoiceCloning"
authors: []
year: 2026
venue: ICLR 2026 submission
arxiv_id: null
pdf_path: "assets/papers/MambaVoiceCloning.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, tts]
created: 2026-05-22
---

# MambaVoiceCloning: SSM-based TTS with Diffusion Control

## 📌 一句话

将 **Mamba（State Space Model）** 引入 TTS——用 SSM 替代 transformer 做 diffusion TTS 的条件路径，在保持表现力的同时显著提升推理效率（SSM 的线性复杂度 vs transformer 的二次复杂度）。

## 🛠 核心方法

**输入 → 输出**: text + speaker prompt → speech

**架构组件**:
1. **Mamba Conditioning Path**: 全 SSM 的条件路径（替代 transformer）
2. **Diffusion Model**: 基于 diffusion 的语音生成
3. **Voice Cloning**: zero-shot speaker cloning

**关键创新**: 首次探索 **SSM-only 的 TTS 条件路径**——证明 Mamba 可以替代 transformer 做 diffusion TTS 的条件编码，推理更快。

## 📊 关键结果 / 评测

- ICLR 2026 submission（anonymous）
- 匿名投稿（ICLR 2026），具体数字见论文 Table 1-3

## 💡 借鉴意义（一句话）

做 TTS 效率优化的人了解——MambaVoiceCloning 证明了 SSM 在 TTS 中替代 transformer 的可行性。

## 🔗 链接

- PDF: [[assets/papers/MambaVoiceCloning.pdf|本地 PDF]]
- 源目录: `TTS-LLM/MAMBAVOICECLONING- EFFICIENT AND EXPRESSIVE TEXT-TO-SPEECH VIA STATE-SPACE MODELING AND DIFFUSION CONTROL.pdf`
