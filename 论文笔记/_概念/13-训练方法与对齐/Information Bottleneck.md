---
type: concept
aliases: [信息瓶颈, IB]
---

# Information Bottleneck

## 定义
一种通过限制表示容量来迫使编码器仅保留任务相关信息的方法。在语音领域常用于解耦不同属性（如 timbre 与 prosody），通过时间压缩、VQ 量化、低维投影等手段实现。

## 数学形式

$$
\min_{p(z|x)} I(X; Z) - \beta \cdot I(Z; Y)
$$

- $I(X; Z)$: 输入与表示的互信息（要压缩）
- $I(Z; Y)$: 表示与目标的互信息（要保留）
- $\beta$: 权衡系数

## 核心要点
1. 在 TTS 中常通过 VQ 量化 + 时间下采样实现，限制 prosody encoder 的容量使其无法编码 content/timbre
2. 与 VAE 的 KL 正则项思路类似，但 IB 更显式地面向任务相关性
3. 瓶颈过窄会丢失有用信息，过宽则无法解耦

## 代表工作
- [[MegaTTS2]]: VQ + 8× 时间压缩作为 prosody 的信息瓶颈
- [[VITS]]: Posterior encoder 的 normalizing flow 也含隐式 IB

## 相关概念
- [[Vector Quantization]]
- [[VAE]]
- [[Mutual Information]]
