---
type: concept
aliases: [Index-TTS]
---

# IndexTTS

## 定义

B站相关团队推出的 zero-shot TTS 系统（Deng et al., 2025），采用 AR Transformer 生成 semantic token + flow matching S2M + 声码器的级联架构。IndexTTS2 的前代系统。

## 核心要点

1. 级联 AR 架构：T2S（AR）→ S2M（Flow Matching）→ Vocoder
2. WER 表现突出，在 LibriSpeech / SeedTTS / AISHELL-1 上均具竞争力
3. 代码开源

## 评测/常见数字

- LibriSpeech test-clean: SS 0.819, WER 3.436%
- SeedTTS test-zh: WER 1.097%（所有基线中最低）

## 代表工作

- [[IndexTTS2]]: 在其基础上增加时长控制和情感表达

## 相关概念

- [[Autoregressive]]
- [[Flow Matching]]
- [[BigVGAN]]
