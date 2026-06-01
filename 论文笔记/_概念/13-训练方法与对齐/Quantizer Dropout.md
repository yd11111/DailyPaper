---
type: concept
aliases: [quantizer dropout, structured dropout for VQ, 量化器丢弃]
---

# Quantizer Dropout

## 定义

训练时对 [[RVQ]] 的量化层数做随机采样（$n_q \sim \text{Uniform}[1, N_q]$），只使用前 $n_q$ 层量化器。本质是 structured dropout 应用于 VQ 层，使单一模型在推理时可以选择任意 $n_q$ 从而实现可变码率。

## 数学形式

$$
n_q \sim \text{Uniform}\{1, 2, \ldots, N_q\}, \quad \hat{y} = \sum_{i=1}^{n_q} Q_i(\text{residual}_i)
$$

推理时固定 $n_q$ 即确定码率 $R = n_q \cdot \text{frame\_rate} \cdot \log_2 N$。

## 核心要点

1. 由 SoundStream 提出，解决"每个码率训练单独模型"的部署成本问题
2. 训练后的 scalable 模型在各码率上匹配甚至超越 bitrate-specific 模型（正则化效应）
3. 由于 RVQ 是加法结构，减少 $n_q$ 不改变 embedding 维度 → encoder/decoder 无需适配
4. 后续被 [[EnCodec]]、[[DAC]] 等沿用

## 代表工作

- [[SoundStream]]: 首次提出（§III-C）
- [[EnCodec]]: 沿用实现 1.5-24 kbps 可伸缩

## 相关概念

- [[RVQ]]
- [[Audio Codec]]
- [[Dropout]]
