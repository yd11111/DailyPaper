---
type: concept
aliases: [VALLE, Vall-E]
---

# VALL-E

## 定义

微软 2023 年提出的"把 TTS 当条件语言建模"的开山之作：以 [[EnCodec]] 离散 codec token 为目标，用 decoder-only Transformer 自回归预测；只需 3 秒参考音频即可零样本克隆音色。

## 数学形式

$$
p(c | x, \tilde{c}) = \prod_{t} p(c_t | c_{<t}, x, \tilde{c})
$$
其中 $c$ 是 EnCodec 离散 token、$x$ 是 phoneme、$\tilde{c}$ 是 prompt token。

## 核心要点

1. **离散 codec token + 大规模 AR LM** 的范式开端；后续 NaturalSpeech 2/3、CosyVoice、Seed-TTS 等都受其启发
2. 多码本：用 AR + NAR 两阶段处理 EnCodec 的多层码本
3. 后续 VALL-E 2、VALL-E X 改进生成稳定性与多语种

## 代表工作

- 原论文 (arXiv 2301.02111, Wang et al. 2023)
- [[VibeVoice]] 在引言中作为前置 TTS LM 范式之一引用

## 评测/常见数字

- LibriSpeech zero-shot WER: ~5.9% (VALL-E base)
- SECS: ~0.58

## 相关概念

- [[EnCodec]]
- [[Audio Codec]]
- [[CosyVoice]]
