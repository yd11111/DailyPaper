---
type: concept
aliases: [Descript Audio Codec, RVQ-GAN]
---

# DAC

## 定义

Descript 2023 年的 Descript Audio Codec：高保真度神经音频压缩，码率 0.5–8 kbps，重建质量在同期 codec 中 SOTA。基于 [[RVQ]] + Mel-GAN 风格判别器。**[[VibeVoice]] 的 acoustic tokenizer 沿用 DAC 的判别器与多 scale loss 设计。**

## 核心要点

1. RVQ-GAN 范式，loss 含 multi-scale STFT + Mel + adversarial
2. 100–800 token/s 区间
3. 是 [[VibeVoice]] 训练目标（loss/disc）的直接来源

## 代表工作

- 原论文 (Kumar et al. 2023, NeurIPS)
- [[VibeVoice]]: 用 DAC loss 设计训练 acoustic tokenizer

## 评测/常见数字

- LibriTTS test-clean PESQ (4 码本/400 token/s): 2.738；UTMOS: 3.433
- 1 码本/100 token/s 时质量明显下降（PESQ 1.246）

## 相关概念

- [[RVQ]]
- [[EnCodec]]
- [[Audio Codec]]
