---
type: concept
aliases: [VoiceFixer]
---

# VoiceFixer

## 定义

Liu et al. (2022) 提出的通用语音恢复系统，将退化语音转换为高质量 44.1kHz 输出。采用分析-恢复-合成三阶段架构。

## 核心要点

1. 三阶段：分析（ResUNet 估计 Mel 频谱）→ 恢复（neural vocoder 合成）→ 后处理
2. 可处理多种退化：噪声、混响、削波、低采样率等
3. 早期语音恢复代表工作，后续被 [[Miipher]]、[[Sidon]] 等基于 SSL 特征的方法超越

## 相关概念

- [[Speech Restoration]]
- [[Miipher]]
- [[Sidon]]
