---
type: concept
aliases: [ConvNeXt-V2, ConvNeXtV2]
domain: LLM
tags: [convnext, cnn-backbone, feature-encoder]
created: 2026-05-29
last_updated: 2026-05-29
---

# ConvNeXt V2

## 定义
纯卷积视觉骨干（Woo et al., CVPR 2023），在 ConvNeXt 基础上引入全卷积掩码自编码预训练 + GRN（Global Response Normalization）。在语音/TTS 里常被借用作轻量级一维特征编码器（如对文本 embedding 做局部上下文建模）。

## 核心要点
1. 纯 CNN，无注意力，参数省、局部建模强。
2. GRN 增强通道间特征竞争。
3. 在 TTS 中通常作为 embedding 后、送入 Transformer 前的局部编码 block。

## 代表工作
- [[PALLE]]: 用一层 ConvNeXt V2 block（dim 1024 + FFN 2048）对文本 token embedding 做编码，再特征维融合进主干 Transformer。
- [[F5-TTS]] / [[E2 TTS]]: 同类用法（ConvNeXt 作文本/输入编码）。

## 相关概念
- [[E2 TTS]]
- [[F5-TTS]]
