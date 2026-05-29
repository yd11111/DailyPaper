---
type: concept
aliases: [时间相似度损失, Temporal Relation Loss]
---

# Time-Relation Loss

## 定义

把高维特征序列 $z^h \in \mathbb{R}^{T \times d_h}$ 和低维特征序列 $z^l \in \mathbb{R}^{T \times d_l}$ 各自的"帧-帧时间相似度矩阵" $\mathbf{G} = z z^\top \in \mathbb{R}^{T \times T}$ 对齐的损失，用于在维度不一致的情况下保持时间结构一致性。由 [[LoSATok]] (Zhang et al., 2026) 提出，灵感来自 neural style transfer 中的 [[Gram Matrix Loss]]。

## 数学形式

$$
\mathcal{L}_{\mathrm{tr}} = \left\| \mathbf{G}^l - \mathrm{sg}(\mathbf{G}^h) \right\|_2
$$

其中 $\mathbf{G}^h = z^h (z^h)^\top$，$\mathbf{G}^l = z^l (z^l)^\top$。实现中通常先对每帧做 L2 归一化，使 $\mathbf{G}$ 是 cosine similarity 矩阵。

## 核心要点

1. **维度无关**：高/低维特征通过 Gram 矩阵投影到相同的 $T \times T$ 空间再对齐，避开了维度不匹配问题。
2. **保留相对结构**：不要求逐维相等，只要求"哪两帧像哪两帧不像"的相对关系一致。
3. **比单纯 reconstruction 更关键**：[[LoSATok]] 消融显示去掉 $\mathcal{L}_{\mathrm{tr}}$ 比去掉 $\mathcal{L}_{\mathrm{recon}}$ 在理解任务上掉得更多（FSC 82.97 vs 86.11）。

## 代表工作

- [[LoSATok]]: 首次在 audio tokenizer 上下文中使用。

## 相关概念

- [[Gram Matrix Loss]]：原始 Gram 损失（图像风格迁移）。
- [[Semantic Bottleneck]]：使用此损失的核心模块。
- [[Knowledge Distillation]]：本质上是一种跨维度的"结构化蒸馏"。
