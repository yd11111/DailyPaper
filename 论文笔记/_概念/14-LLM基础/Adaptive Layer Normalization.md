---
type: concept
aliases: [AdaLN, Adaptive LayerNorm]
---

# Adaptive Layer Normalization

## 定义

在标准 LayerNorm 基础上引入条件参数（scale $a$ 和 shift $b$），使归一化结果可根据外部条件信号动态调整。常用于 Diffusion Transformer（DiT）和条件生成模型中注入时间步、类别或阶段信息。

## 数学形式

$$
\mathrm{AdaLN}(h, c) = a_c \cdot \mathrm{LayerNorm}(h) + b_c
$$

- $h$: 隐藏状态
- $c$: 条件信号（如时间步、stage index）
- $a_c, b_c$: 由条件 $c$ 通过线性层生成的 scale 和 shift 参数

## 核心要点

1. 比 cross-attention 更轻量的条件注入方式
2. 在 DiT 中替代传统的 class-conditional injection
3. VALL-E 的 NAR 模型用 AdaLN 注入 RVQ stage index $i$

## 代表工作

- [[VALL-E]]: NAR 模型用 AdaLN 注入 codebook stage 信息
- DiT (Peebles & Xie 2023): 在 Diffusion Transformer 中首次系统验证 AdaLN

## 相关概念

- [[Transformer]]
- [[DiT]]
