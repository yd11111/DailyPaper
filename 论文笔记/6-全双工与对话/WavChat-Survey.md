---
title: "WavChat: A Survey of Spoken Dialogue Models"
method_name: "WavChat-Survey"
authors: [WavChat Team]
year: 2024
venue: arXiv
arxiv_id: "2411.13577"
pdf_path: "assets/papers/WavChat-Survey.pdf"
library_source: "高德文献库"
source_topic: "Survey"
tags: [classic, survey, full-duplex]
created: 2026-05-22
---

# WavChat: Spoken Dialogue Models 综述

## 📌 一句话

目前最全面的**语音对话模型综述**，梳理了从 cascade（ASR→LLM→TTS）到 end-to-end spoken dialogue model 的技术演进，覆盖架构范式、多阶段训练、对齐策略等维度。

## 🛠 核心方法

**输入 → 输出**: 综述论文，无独立模型

**架构组件**（按分类维度）:
1. **Architecture Paradigms**: cascade / text-centric / speech-centric / fully end-to-end
2. **Multi-stage Training**: pre-train → SFT → alignment，各阶段的 speech 特殊处理
3. **Alignment Post-training**: DPO / RLHF 在语音对话场景的应用

**关键创新**: 首次系统性地梳理了 spoken dialogue model 的完整技术栈，从数据、模型、训练、评测到部署。

## 🖼 架构图

![Figure 2: General overview of spoken dialogue systems — 四类架构范式分类](https://ar5iv.labs.arxiv.org/html/2411.13577/assets/images/img2-method.png)

## 📊 关键结果 / 评测

- 综述，无实验结果
- 覆盖 2023-2024 年所有主流 spoken dialogue 模型

## 💡 借鉴意义（一句话）

做全双工对话 / Speech LLM 的人**入门必读 survey**——快速了解从 GPT-4o 到开源 spoken LM 的全景。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2411.13577
- PDF: [[assets/papers/WavChat-Survey.pdf|本地 PDF]]
- 源目录: `Survey/Wavchat survey.pdf`
