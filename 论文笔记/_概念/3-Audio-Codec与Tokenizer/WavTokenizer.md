---
type: concept
aliases: []
---

# WavTokenizer

## 定义

上海交大 2024 年的单码本低码率 codec：$N_q=1$、40 或 75 Hz 帧率，专为 audio LM 减少序列长度而设计。

## 核心要点

1. 单码本：避免 RVQ 多层展开
2. 帧率可低至 40 Hz；UTMOS 高（test-clean 4.05）
3. PESQ/STOI 在低码率下仍偏弱

## 代表工作

- 原论文 (Ji et al. 2025, ICLR)
- [[VibeVoice]] tokenizer 重建对比基线

## 评测/常见数字

- 75 Hz 配置：LibriTTS test-clean PESQ 2.373 / UTMOS 4.049
- 40 Hz 配置：PESQ 1.703 / UTMOS 3.602

## 相关概念

- [[Audio Codec]]
- [[EnCodec]]
- [[SpeechTokenizer]]
