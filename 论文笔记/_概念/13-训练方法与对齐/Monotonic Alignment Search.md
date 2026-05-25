---
type: concept
aliases: [MAS]
---

# Monotonic Alignment Search

## 定义

VITS 中提出的动态规划算法，在文本-语音 alignment 中搜索最优的单调对齐路径，使模型无需外部 forced alignment 工具即可学习 duration。

## 核心要点

1. 在 VITS 的 normalizing flow 框架下，通过最大化 log-likelihood 搜索最优 alignment
2. 约束: 对齐必须单调递增（phoneme 顺序不变）
3. 替代了 FastSpeech 2 依赖的外部 MFA 工具
4. 使 TTS 训练更加端到端

## 代表工作

- [[VITS]]: 首次提出
- [[NaturalSpeech]]: 沿用 MAS

## 相关概念

- [[Forced Alignment]]
- [[Montreal Forced Alignment]]
- [[Duration Predictor]]
- [[Length Regulator]]
