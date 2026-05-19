---
type: concept
aliases: []
---

# DDPM

## 定义

Denoising Diffusion Probabilistic Model，Ho 2020。前向 $\mathbf{p}_t = \sqrt{\bar\alpha_t}\mathbf{p}_0 + \sqrt{1-\bar\alpha_t}\epsilon$，反向用 U-Net/DiT 预测 $\epsilon$。SemaVoice 的 LocDiT 用此目标。

## 核心要点

1. （待补充）

## 代表工作

- [[SemaVoice]]

## 相关概念

[[DiT]] / [[Classifier-Free Guidance]]
