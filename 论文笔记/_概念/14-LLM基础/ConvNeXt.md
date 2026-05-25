---
type: concept
aliases: [ConvNeXt V2, ConvNeXt-V2]
---

# ConvNeXt

## 定义

Meta 提出的现代化 CNN 架构，用纯卷积块对标 Transformer 性能。在 TTS 中常用于局部特征精炼（如 F5-TTS 用 ConvNeXt V2 处理字符序列，让文本信息通过局部卷积"扩散"到邻近 filler token 位置）。

## 核心要点

1. 每个 block：Depthwise Conv (7x7) → LayerNorm → 1x1 Conv (expand) → GELU → 1x1 Conv (contract)
2. V2 版本加入 GRN (Global Response Normalization) 提升特征多样性
3. 在 F5-TTS 中用 4 层 ConvNeXt V2 (512 dim, 1024 FFN) 精炼填充后的字符序列

## 代表工作

- [[F5-TTS]]: 用 ConvNeXt V2 替代直接拼接，解决 E2 TTS 的语义-声学纠缠问题

## 相关概念

- [[DiT]]
- [[F5-TTS]]
