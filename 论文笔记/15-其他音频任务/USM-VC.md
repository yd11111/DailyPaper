---
title: "USM-VC: Mitigating Timbre Leakage with Universal Semantic Mapping Residual Block for Zero-Shot Voice Conversion"
method_name: "USM-VC"
authors: []
year: 2025
venue: arXiv
arxiv_id: "2504.08524"
pdf_path: "assets/papers/USM-VC.pdf"
library_source: "高德文献库"
source_topic: "VC"
tags: [classic, vc]
created: 2026-05-22
---

# USM-VC: Universal Semantic Mapping for Voice Conversion

## 📌 一句话

提出 **USM 残差模块**解决 zero-shot VC 中的 timbre leakage 问题——在 content encoder 和 conversion model 之间加入 semantic mapping 层，将说话人相关的 timbre 信息从 content 表示中剥离。

## 🛠 核心方法

**输入 → 输出**: source speech + target speaker prompt → converted speech

**架构组件**（按数据流顺序）:
1. **Content Encoder (PPG/HuBERT)**: 提取源语音的内容特征
2. **USM Residual Block**: 通过残差映射剥离 timbre leakage
3. **LM-based Conversion / Diffusion Decoder**: 基于 LM 或 diffusion 做语音转换
4. **Target Speaker Encoder**: 从目标说话人 prompt 提取 timbre embedding

**关键创新**: 直接在 content representation 层面用 residual block 做 timbre 去相关，而非靠对抗训练或信息瓶颈——更简单稳定。

## 🖼 架构图

![Figure 1: USM residual block](https://ar5iv.labs.arxiv.org/html/2504.08524/assets/usm2.png)

## 📊 关键结果 / 评测

- 首页未给出具体数字，待全文确认

## 💡 借鉴意义（一句话）

做 VC / 声音克隆的人关注——timbre leakage 是 VC 领域的核心难题，USM 的残差模块方案简洁有效。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2504.08524
- PDF: [[assets/papers/USM-VC.pdf|本地 PDF]]
- 源目录: `VC/USM-VC.pdf`
