---
type: concept
aliases: [Composable and Controllable Emotional TTS]
domain: TTS
tags: [emotion-control, activation-steering, mixed-emotion]
created: 2026-07-03
---

# CoCoEmo

## 定义

Composable and Controllable Human-like Emotional TTS via Activation Steering（Wang et al., 2026）。在 TTS 的 SLM（语言模型）阶段提取情感方向向量并通过加权组合实现混合情感合成，无需重新训练。

## 核心要点

1. 在 SLM 的 attention output 激活（last-token 位置）提取情感-neutral 差向量
2. 按目标情感比例加权求和，推理时注入指定层实现混合情感控制
3. 从 multi-rater annotation disagreement 构建混合情感 ground truth

## 代表工作

- Wang et al. (2026), arXiv:2602.03420: 原始论文
- [[GeometricEmotionSteering]]: 以 CoCoEmo 的 SLM steering 方法为基础之一，做几何分析对比

## 相关概念

- [[Activation Steering]]
- [[EmoSteer-TTS]]
- [[CosyVoice 2]]
