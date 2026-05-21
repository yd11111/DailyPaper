---
type: concept
aliases: [Stable Audio Open, Stable Audio 2]
---

# Stable Audio

## 定义

Stability AI 2023–2024 的商业级 [[Text-to-Audio]] 模型族（Evans et al.）。在 audio VAE 上用 [[Flow Matching]] / [[DiT]] 做条件生成，最大特点是支持**长 form**（最长 ~3 分钟）的 stereo 44.1 kHz 音频生成，目标是音乐 / 音效专业用户。Stable Audio Open 是其开源版本。

## 核心要点

1. **stereo 44.1 kHz**：少有的同时做高采样率 + 长时长的 TTA 模型
2. **audio VAE + DiT + Flow Matching**：latent 扩散的现代化栈
3. **长 form 一致性**：通过 timing 条件化让模型对 duration 敏感
4. **商业模型** vs **Stable Audio Open**：后者完全开源、训练数据可追溯
5. 与 [[AudioLDM]] / [[Tango]] 同为开源 TTA 三大代表

## 代表工作

- Stable Audio 1.0 报告 (arXiv 2310.10103)
- Stable Audio 2 (2024)
- Stable Audio Open (arXiv 2407.14358)
- [[WavFlow]]：直接对比的目标（waveform-space vs latent-space generation）

## 评测/常见数字

- AudioCaps FAD：Stable Audio Open ≈ 1.3
- 支持 44.1 kHz stereo，up to ~3 min

## 相关概念

- [[AudioLDM]]
- [[Audio VAE]]
- [[Flow Matching]]
- [[DiT]]
- [[Text-to-Audio]]
