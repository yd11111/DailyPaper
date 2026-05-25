---
type: concept
aliases: [Tacotron2, Tacotron]
---

# Tacotron 2

## 定义

Google 提出的经典自回归（AR）TTS 模型，使用 attention-based seq2seq 架构将文本映射到 Mel-Spectrogram，再通过 WaveNet vocoder 合成波形。是早期端到端 TTS 的里程碑。

## 核心要点

1. Encoder-Decoder + Location-Sensitive Attention 架构
2. AR 逐帧生成 Mel-Spectrogram，通过 stop token 判断结束
3. 合成质量接近人类水平（MOS ~4.5 on 单说话人），但推理慢
4. 经典痛点：attention alignment 不稳定导致跳词、重复、漏词

## 代表工作

- Tacotron (Wang et al., 2017): 初版
- Tacotron 2 (Shen et al., 2018): 改进版，搭配 WaveNet vocoder
- [[VALL-E]]: 将 TTS 从 Mel 回归转向 codec language modeling

## 评测/常见数字

- 单说话人 MOS ~4.5（LJSpeech / 内部数据集）
- 推理较慢（AR 逐帧 + WaveNet vocoder）

## 相关概念

- [[Mel-Spectrogram]]
- [[Autoregressive Model]]
- [[FastSpeech 2]]
- [[Phoneme]]
