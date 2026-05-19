---
type: concept
aliases: [Neural Audio Codec, 神经音频编解码器]
---

# Audio Codec

## 定义

把原始 waveform 压成低维表示（连续或离散）再还原回波形的神经网络。现代 codec 是 audio LM / TTS 的语音 token 来源。

## 核心要点

1. **离散 codec**（[[EnCodec]]、[[DAC]]、[[SpeechTokenizer]]、[[WavTokenizer]]）：基于 [[RVQ]]，输出整数 token 序列
2. **连续 codec / VAE-style**（[[VibeVoice]] 的 σ-VAE acoustic tokenizer）：输出连续 latent，可直接喂扩散/流匹配头
3. 关键指标：码率 (token/s 或 bps)、PESQ、STOI、UTMOS
4. **越低的 token rate 越利于 LLM 长上下文建模**

## 代表工作

- [[EnCodec]] / [[DAC]] / [[SpeechTokenizer]] / [[WavTokenizer]]
- [[VibeVoice]]: 单维连续 σ-VAE，7.5 Hz / 1 token

## 相关概念

- [[RVQ]]
- [[Acoustic Tokenizer]]
- [[Semantic Tokenizer]]
