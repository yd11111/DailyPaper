---
type: concept
aliases: [双层语义监督]
---

# Dual-Level Semantic Supervision

## 定义

在训练 unified audio tokenizer 时，同时用高维（例如 1280-d MiDashengLM）和低维（例如 SemBo 压缩出的 128-d）两套语义信号约束 acoustic encoder 的输出，让低维 latent 同时承载语义和声学信息。由 [[LoSATok]] (Zhang et al., 2026) 提出。

## 数学形式

$$
\mathcal{L}_H = \left\| z_a^h - \mathrm{sg}(z_s^h) \right\|_2, \quad \mathcal{L}_L = \left\| z_a^l - \mathrm{sg}(z_s^l) \right\|_2
$$

LoSATok 中二者 1:1 等权，整体乘 $\lambda_{\mathrm{sem}}=45$ 加入总训练目标。

## 核心要点

1. **高维信号给完整语义目标**：1280-d 给 acoustic encoder 一个"完整语义画像"作隐式监督。
2. **低维信号是直接约束目标**：128-d 是最终 unified latent 实际所在空间的直接 supervision。
3. **两者互补**：LoSATok 消融显示 **去掉低维监督几乎让 unified latent 完全失去理解能力**（FSC 从 59.87 暴跌到 6.30），证明 low-dim 监督是这套设计的灵魂。
4. **不可被 training-free 替代**：用 Channel Merging 替代 SemBo 提供低维监督，理解性能依然崩溃（FSC 5.11），说明 dual-level 中低维项不仅要"是低维"，还要"是学过的低维"。

## 代表工作

- [[LoSATok]]: 首次提出，是其 unified tokenizer 设计的核心。

## 相关概念

- [[Semantic Bottleneck]]：提供 low-dim semantic target $z_s^l$。
- [[Knowledge Distillation]]：本质是 multi-level distillation。
- [[Mixed Token]]：dual-level 输出的 unified latent 即语义+声学混合表示。
