---
type: concept
aliases: [LongCat-AudioDiT, LongCat Audio DiT]
domain: TTS
tags: [tts, diffusion-transformer, continuous-latent, large-scale]
created: 2026-06-09
related_maps:
  - "[[TTS-技术路线图]]"
---

# LongCat-Audio-DiT

## 定义

大规模连续 latent Diffusion Transformer TTS 系统（3.5B 参数），在 Seed-TTS-Eval 上取得了当时开源模型中最高的 speaker similarity（SIM 78.6/81.8 EN/ZH）。

## 核心要点

1. 3.5B 参数的连续 latent DiT 架构
2. 在 Seed-TTS-Eval 上 SIM 指标在开源模型中领先
3. WER 1.50 (EN) / CER 1.09 (ZH) 也表现优秀

## 代表工作

- LongCat-Audio-DiT (2026)

## 相关概念

- [[Diffusion Transformer]]
- [[Conditional Flow Matching]]
- [[VoxCPM2]]
