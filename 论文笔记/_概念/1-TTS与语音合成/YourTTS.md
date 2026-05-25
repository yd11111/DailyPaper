---
type: concept
aliases: [Your-TTS]
---

# YourTTS

## 定义

基于 VITS 架构的多语种零样本 TTS 模型，通过 speaker encoder 提取说话人表征实现 zero-shot voice cloning。是 VALL-E 论文中的主要对比 baseline。

## 核心要点

1. 基于 VITS 的端到端架构 + speaker encoder
2. 支持多语种（英文 + 葡萄牙语等）
3. 使用 speaker encoding 方式实现 zero-shot，与 VALL-E 的 in-context learning 方式不同
4. 在域外（unseen speaker）场景质量退化明显

## 代表工作

- YourTTS (Casanova et al., 2022): 原始论文
- [[VALL-E]]: 在 LibriSpeech / VCTK 上显著超越 YourTTS
- [[VALL-E-X]]: 在跨语言 TTS 上以 YourTTS 为基线（ASV-Score 0.30 vs 0.36）

## 评测/常见数字

- LibriSpeech: WER 7.7%, SPK similarity 0.337, SMOS 3.45
- VCTK (108 speakers): SPK 0.357 (3s prompt)
- 被 VALL-E 在所有指标上大幅超越

## 相关概念

- [[VITS]]
- [[Zero-shot TTS]]
- [[VALL-E]]
