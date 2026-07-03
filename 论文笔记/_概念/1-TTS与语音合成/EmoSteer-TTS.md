---
type: concept
aliases: [EmoSteer]
domain: TTS
tags: [emotion-control, activation-steering, cfm-steering]
created: 2026-07-03
---

# EmoSteer-TTS

## 定义

Fine-grained and Training-free Emotion-Controllable Text-to-Speech via Activation Steering（Xie et al., 2025）。在 TTS 的 CFM（Conditional Flow Matching）阶段提取情感方向向量，通过 residual stream 激活的 L2 归一化 + emotion classifier mask 实现连续单情感强度控制。

## 核心要点

1. 在 CFM 的 residual stream 激活上提取方向向量，L2 归一化后用情感分类器 mask 出 top-k 情感相关帧
2. 实现连续的单情感强度控制（通过调节 $\alpha$）
3. 无需重新训练模型

## 代表工作

- Xie et al. (2025), arXiv:2508.03543: 原始论文
- [[GeometricEmotionSteering]]: 以 EmoSteer-TTS 的 CFM steering 方法为基础之一，做几何分析对比

## 相关概念

- [[Activation Steering]]
- [[CoCoEmo]]
- [[Conditional Flow Matching]]
- [[CosyVoice 2]]
