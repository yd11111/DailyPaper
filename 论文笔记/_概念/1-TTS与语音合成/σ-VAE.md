---
type: concept
aliases: []
---

# σ-VAE

## 定义

VAE 变体：方差 $\boldsymbol\sigma$ 不再由 encoder 输出，而是从预定义分布 $\mathcal{N}(0, C_\sigma)$ 采样，保证 latent 方差非消失，给下游 AR 一个稳定的目标。

## 核心要点

1. （待补充）

## 代表工作

- [[SemaVoice]]

## 相关概念

[[VAE]] / [[KL Divergence]]
