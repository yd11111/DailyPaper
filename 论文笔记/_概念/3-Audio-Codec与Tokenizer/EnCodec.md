---
type: concept
aliases: [Encodec]
---

# EnCodec

## 定义

Meta 2022 年的神经音频编解码器 (Défossez et al.)。基于 [[RVQ]] 的多层离散 codec，支持 1.5–24 kbps；24 kHz 输入下典型帧率 75 Hz × 8 码本 = 600 token/s。是 [[VALL-E]] 等离散 token TTS 的"事实上"基础 codec。

## 数学形式

编码：encoder → RVQ → bitstream；总 token 数 $= \text{frame\_rate} \times N_q$，其中 $N_q$ 是码本数。

## 核心要点

1. **RVQ 多层量化**：层数 $N_q$ 决定码率与质量
2. 重建质量好但码率高 (300–600 token/s) → AR LM 建长音频代价大
3. [[VibeVoice]] 把它当压缩对比基线：声称比 EnCodec 8 层 (600 token/s) 压缩 80×（→ 7.5 token/s）

## 代表工作

- 原论文 (arXiv 2210.13438)
- [[VALL-E]]、[[VibeVoice]] 等都把它列为基线

## 评测/常见数字

- LibriTTS test-clean PESQ (8 码本/600 token/s): 2.72；STOI: 0.939
- LibriTTS test-other PESQ: 2.682；UTMOS: 2.657

## 相关概念

- [[RVQ]]
- [[DAC]]
- [[Audio Codec]]
