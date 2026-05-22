---
title: "Dynamic-SUPERB Phase-2: A Collaboratively Expanding Benchmark for Measuring the Capabilities of Spoken Language Models"
method_name: "Dynamic-SUPERB"
authors: [Dynamic-SUPERB Team]
year: 2024
venue: arXiv
arxiv_id: "2411.05361"
pdf_path: "assets/papers/Dynamic-SUPERB.pdf"
library_source: "高德文献库"
source_topic: "SSL"
tags: [classic, ssl, eval]
created: 2026-05-22
---

# Dynamic-SUPERB: Expandable Spoken Language Model Benchmark

## 📌 一句话

[[SUPERB]] 的进化版——从固定 10 任务扩展为**社区协作式动态 benchmark**，覆盖数百个语音理解任务，专门用于评测 Spoken Language Models（如 GSLM / AudioPaLM 等）。

## 🛠 核心方法

**输入 → 输出**: 评测框架，无独立模型

**架构组件**（按设计维度）:
1. **社区驱动**: 任何人可提交新评测任务，benchmark 持续扩展
2. **多维度覆盖**: 语音内容理解 / 说话人属性 / 环境声 / 语义推理等
3. **Spoken LM Focus**: 评测对象从 SSL encoder 扩展到 generative spoken LM
4. **Instruction Following**: 部分任务需要模型理解自然语言指令

**关键创新**: 从 SUPERB 的"固定任务集"升级为"动态扩展"模式，适应 Speech LLM 时代模型能力爆发式增长的评测需求。

## 📊 关键结果 / 评测

- Phase-2 覆盖数百个任务，具体数字待全文确认
- 目标：持续追踪 spoken LM 的能力前沿

## 💡 借鉴意义（一句话）

做 Speech LLM / Audio LLM 评测的人关注——Dynamic-SUPERB 是目前最全面的 spoken LM 评测框架。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2411.05361
- PDF: [[assets/papers/Dynamic-SUPERB.pdf|本地 PDF]]
- 源目录: `SSL/SUPERB2.pdf`
