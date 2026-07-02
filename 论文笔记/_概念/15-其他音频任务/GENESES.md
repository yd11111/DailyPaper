---
type: concept
aliases: [GENESES]
---

# GENESES

## 定义

Asai et al. (2026) 提出的联合语音分离与恢复模型，使用 MMDiT (Multi-Modal Diffusion Transformer) + DACVAE 架构，通过 flow matching 在 VAE 隐空间生成分离+恢复后的多说话人音频。参数量约 393M。

## 核心要点

1. 首个尝试在统一框架内同时做分离和恢复的模型之一
2. 主要针对干净独白的人工混合开发，在真实对话场景上效果有限（WER 可达 80%+）
3. 使用 MMDiT 三模态架构（SSL 条件 + 两个说话人 VAE latent）
4. 推理使用 ODE solver，100 步，RTF 约 0.6，较慢
5. [[DialogueSidon]] 的主要对比基线

## 代表工作

- Asai et al. 2026: GENESES 原始论文
- [[DialogueSidon]]: 在对话场景上大幅超越 GENESES

## 相关概念

- [[Source Separation]]
- [[Speech Restoration]]
- [[DiT]]
- [[DAC]]
