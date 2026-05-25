---
type: concept
aliases: [韵律迁移, 风格迁移, Style Transfer for TTS]
---

# Prosody Transfer

## 定义
将参考语音的韵律风格（pitch 轮廓、节奏、能量模式、情感语调）迁移到目标说话人的合成语音上，同时保持目标说话人的音色（timbre）不变。

## 核心要点
1. 核心难点在于 prosody 和 timbre 的解耦——朴素方法常导致 timbre 泄漏
2. 评估指标通常包含 pitch 分布统计量（σ 标准差、γ 偏度、κ 峰度）、Duration Error (DE)、SIM（timbre 保真度）
3. 常见方法：reference encoder（GST/VAE）、gradient reversal（Daft-Exprt）、VQ 解耦（Mega-TTS 2）

## 代表工作
- [[MegaTTS2]]: Prosody interpolation——在 P-LLM 概率空间做 soft mixing
- CopyCat: Reference encoder 捕获时序韵律表示
- Daft-Exprt: 梯度反转惩罚说话人身份泄漏

## 相关概念
- [[Information Bottleneck]]
- [[MRTE]]
- [[Speaker Encoder]]
