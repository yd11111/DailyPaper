---
type: concept
aliases: [Adafactor Optimizer]
---

# Adafactor

## 定义
Google 提出的内存高效自适应优化器，通过将二阶矩矩阵分解为行和列因子来减少内存占用（从 O(mn) 降至 O(m+n)），同时支持自适应学习率，适合训练大型 Transformer 模型。

## 核心要点
1. 不存储完整的二阶矩，而是用行/列因子近似，内存节省约 50%
2. 支持 inverse square-root learning rate schedule
3. 是 T5 系列模型的默认优化器
4. 相比 Adam 在大模型上内存更友好，性能相当

## 代表工作
- Shazeer & Stern (2018): "Adafactor: Adaptive Learning Rates with Sublinear Memory Cost"
- [[SPEAR-TTS]]: S1 和 S2 训练均使用 Adafactor + inverse square-root decay

## 相关概念
- [[T5]]
- [[Transformer]]
