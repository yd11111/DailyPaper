---
title: "Zero-Shot Voice Conversion with Diffusion Transformers"
method_name: "Seed-VC"
authors: []
year: 2024
venue: arXiv
arxiv_id: "2411.09943"
pdf_path: "assets/papers/Seed-VC.pdf"
library_source: "高德文献库"
source_topic: "VC"
tags: [classic, vc]
created: 2026-05-22
---

# Seed-VC: Diffusion Transformer for Voice Conversion

## 📌 一句话

字节 Seed 团队提出基于 **Diffusion Transformer (DiT)** 的 zero-shot 声音转换系统，用 U-Net style skip connections + timbre shifter 实现高质量音色转换，同时保持内容和韵律。

## 🛠 核心方法

**输入 → 输出**: source speech + reference timbre audio → converted speech

**架构组件**（按数据流顺序）:
1. **Content Encoder**: 提取源语音的内容+韵律特征（去 timbre）
2. **Timbre Encoder**: 从 reference audio 提取目标 timbre embedding
3. **Diffusion Transformer**: U-Net style DiT，以 content + timbre 为条件做去噪生成
4. **Timbre Shifter**: 训练时随机替换 timbre prompt，学习 timbre 解耦

**关键创新**: 把 diffusion 从 U-Net 升级到 **DiT**（Diffusion Transformer），在 VC 场景下利用 attention 的长程依赖能力提升时长对齐和细节保持。

## 🖼 架构图

![Figure 2: Seed-VC training pipeline — content/timbre 分离 + diffusion transformer](https://ar5iv.labs.arxiv.org/html/2411.09943/assets/SeedVC-train.png)

## 📊 关键结果 / 评测

- 首页未给出具体数字，待全文确认
- 目标: zero-shot VC with high speaker similarity and content preservation

## 💡 借鉴意义（一句话）

做 VC / TTS 的人关注——Seed-VC 展示了 DiT 在 VC 场景的优势，其 timbre shifter 训练策略可迁移到 TTS。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2411.09943
- PDF: [[assets/papers/Seed-VC.pdf|本地 PDF]]
- 源目录: `VC/seedVC.pdf`
