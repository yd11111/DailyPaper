---
title: "SpeechAlign: Aligning Speech Generation to Human Preferences"
method_name: "SpeechAlign"
authors: [Dong Zhang, Zhaowei Li, Shimin Li, Xin Zhang, Pengyu Wang, Yaqian Zhou, Xipeng Qiu]
year: 2024
venue: ACL 2024
arxiv_id: "2404.05600"
pdf_path: "assets/papers/SpeechAlign.pdf"
library_source: "高德文献库"
source_topic: "RLHF"
tags: [classic, rlhf, tts]
created: 2026-05-22
---

# SpeechAlign: Speech Generation Aligned to Human Preferences

## 📌 一句话

复旦提出将 LLM alignment（RLHF/DPO）范式迁移到 codec language model TTS——通过自动构建 preference pairs（好/差合成样本）+ DPO 训练，无需人工标注即可提升 TTS 的自然度和说话人相似度。

## 🛠 核心方法

**输入 → 输出**: codec LM TTS → aligned codec LM TTS (with DPO)

**架构组件**（按数据流顺序）:
1. **Codec LM Baseline**: AR + NAR codec language model（类 VALL-E 架构）
2. **Preference Data Construction**: 同一文本多次采样 → 用 MOS predictor + speaker similarity 自动标注 win/lose pairs
3. **DPO Training**: 直接偏好优化，让 LM 偏向高质量合成结果
4. **无人工标注**: 整个流程全自动，preference signal 来自现有评估模型

**关键创新**: 首次在 TTS 领域完整走通 DPO pipeline——关键洞察是 TTS 可以用现有 MOS predictor 做自动 reward，不需要像 text LLM 那样依赖人工标注。

## 🖼 架构图

![Figure 3: SpeechAlign — codec LM inference + preference data collection + DPO optimization](https://ar5iv.labs.arxiv.org/html/2404.05600/assets/Figures/speechalign.png)

## 📊 关键结果 / 评测

- 首页未给出具体数字，待全文确认
- 目标：用 DPO 提升 codec LM TTS 的 MOS 和 speaker similarity

## 💡 借鉴意义（一句话）

做 TTS RLHF / alignment 的人**必读**——SpeechAlign 证明了 codec LM TTS 可以用纯自动化 preference pipeline 做 DPO，是 [[GSRM]] 等后续工作的先驱。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2404.05600
- PDF: [[assets/papers/SpeechAlign.pdf|本地 PDF]]
- 源目录: `RLHF/SpeechAlign.pdf`
