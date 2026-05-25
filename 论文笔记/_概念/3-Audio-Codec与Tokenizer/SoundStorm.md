---
type: concept
aliases: [SoundStorm]
---

# SoundStorm

## 定义
Google 提出的高效音频生成模型，使用并行化的 masked language modeling 从语义 token 生成声学 token，速度显著快于自回归方法。

## 核心要点
1. 使用 MaskGIT 风格的迭代并行解码，而非逐 token 自回归
2. 以语义 token（来自 AudioLM 的 w2v-BERT）为条件，生成 SoundStream 的多层 RVQ token
3. 从粗到细（coarse-to-fine）逐层生成，每层内并行
4. 相比自回归 AudioLM 快 100 倍以上

## 代表工作
- [[GPT-SoVITS]]: 受 SoundStorm 的 token 建模思路影响
- [[VALL-E]]: 同期的 AR 语义 token TTS 方案

## 相关概念
- [[RVQ]]
- [[Semantic Token]]
- [[Acoustic Token]]
