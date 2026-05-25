---
type: concept
aliases: [SoundStorm]
---

# SoundStorm

## 定义
Google 提出的非自回归 acoustic token 生成模型，用 MaskGIT 范式并行化 AudioLM 的 acoustic stage（Stage 2 + 3），实现数百倍加速。

## 核心要点
1. 替代 AudioLM 中串行的 coarse/fine acoustic 自回归生成
2. 使用 masked prediction + iterative refinement，在 RVQ 层间从粗到细逐层并行生成
3. 以 semantic token 为条件，保持语义连贯性
4. 比 AudioLM acoustic stages 快约 100x

## 代表工作
- [[AudioLM]]: SoundStorm 的前置工作，提供 semantic + acoustic 分层框架

## 相关概念
- [[AudioLM]]
- [[RVQ]]
- [[Acoustic Token]]
- [[Semantic Token]]
