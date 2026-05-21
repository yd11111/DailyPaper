---
type: concept
aliases: [Raon Core]
---

# Raon-OpenTTS-Core

## 定义

[[Raon-OpenTTS-Pool]]（615K h）经过三指标 Combined 15% 分位过滤后得到的 **510K 小时、194M 段**高质量训练子集，是 [[Raon-OpenTTS]] 0.3B / 1B 模型的实际训练数据。

## 核心要点

1. **过滤策略**: [[DNSMOS]] + [[Whisper]] WER + [[Speech Ratio]] 三指标各自 rank → 平均 rank → 砍最差 15%
2. **阈值**: DNSMOS < 2.24 / SR < 0.79 / WER > 0.35
3. **整体保留率**: 84.7%
4. **过滤效果**: 在 [[Seed-TTS-eval]] 上 WER 从 2.19（不过滤）降到 2.00（Combined 15%）；更激进的 50% 反而恶化
5. **结论**: TTS 训练**质量重要但数量更重要**，温和过滤优于激进过滤

## 各数据集保留率

| 来源 | 保留率 |
|---|---|
| [[LibriTTS]]-R / [[HiFiTTS2]] / LibriHeavy / [[Emilia]] | 93-98% |
| [[Raon-YouTube-Commons]] / Emilia-YODAS | 83-88% |
| [[VoxPopuli]] | 71.8% |
| People's Speech (Dirty) | 48.2% |

## 代表工作

- [[Raon-OpenTTS]]: 发布与训练
- [[F5-TTS]]: 用同款 [[DiT]] 架构但只训过 [[Emilia]]，是 Raon 数据效应对照

## 相关概念

- [[Raon-OpenTTS-Pool]]: 母集
- [[DNSMOS]]、[[Speech Ratio]]、[[Whisper]]: 过滤指标
