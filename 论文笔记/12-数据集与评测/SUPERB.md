---
title: "SUPERB: Speech processing Universal PERformance Benchmark"
method_name: "SUPERB"
authors: [Shu-wen Yang, Po-Han Chi, Yung-Sung Chuang, Cheng-I Jeff Lai, Kushal Lakhotia]
year: 2021
venue: Interspeech 2021
arxiv_id: "2105.01051"
pdf_path: "assets/papers/SUPERB.pdf"
library_source: "高德文献库"
source_topic: "SSL"
tags: [classic, ssl, eval]
created: 2026-05-22
---

# SUPERB: Speech Processing Universal Benchmark

## 📌 一句话

台大 + Meta 发布的**语音 SSL 模型统一评测框架**，覆盖 ASR / KS / SID / SV / SD / IC / SF / SE / SS / ST 10 个下游任务，是评估 SSL 语音表示质量的标准 benchmark。

## 🛠 核心方法

**输入 → 输出**: 评测框架，无独立模型

**架构组件**（按设计维度）:
1. **10 Downstream Tasks**: PR / KS / SID / SV / SD / IC / SF / SE / SS / ST
2. **Frozen SSL + Linear Probe**: 统一评测协议——冻结 SSL 模型，只训练轻量下游 head
3. **Weighted Sum of Layers**: 学习各层权重的加权和作为最终表示
4. **Leaderboard**: 公开排行榜持续接收新模型提交

**关键创新**: 首次为语音 SSL 建立了**统一 + 公平**的评测标准——之前各模型各自挑有利任务报告，SUPERB 强制全任务评测消除了 cherry-picking。

## 📊 关键结果 / 评测

- 初始评测: HuBERT-Large 在多数任务上领先
- 后续: WavLM 在 SUPERB leaderboard 上刷新多项记录

## 💡 借鉴意义（一句话）

做语音 SSL 的人**必须了解**——SUPERB 是发表 SSL 论文时的标准评测要求，不跑 SUPERB 基本不可能被顶会接收。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2105.01051
- PDF: [[assets/papers/SUPERB.pdf|本地 PDF]]
- 源目录: `SSL/SUPERB.pdf`
