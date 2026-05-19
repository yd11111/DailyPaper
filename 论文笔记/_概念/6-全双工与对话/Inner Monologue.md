---
type: concept
aliases: [Inner Monologue, 内心独白]
---

# Inner Monologue

## 定义
[[Moshi]] 提出的训练技巧，让模型先生成内部的 "想法" 文本 token，再生成对应的 speech token。

## 核心要点
1. 缓解 speech-only 模型的语义弱化问题
2. 类似 chain-of-thought 但用于 speech-text 联合生成
3. 和 acoustic delay 一起让 multi-stream 并行 AR 可行

## 代表工作
- [[Moshi]] 提出，[[OmniFlatten]] 视其为复杂设计并选择 flatten 简化

## 相关概念
- [[OmniFlatten]]
