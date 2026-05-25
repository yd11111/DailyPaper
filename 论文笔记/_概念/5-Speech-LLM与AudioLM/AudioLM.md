---
type: concept
aliases: [Audio LM, Audio Language Model]
---

# AudioLM

## 定义

Google 提出的三段式离散音频语言模型，将音频生成分为 semantic token -> coarse acoustic token -> fine acoustic token 三阶段，实现高质量的语音和音乐续写。是 speech-to-speech 模型，不接受文本输入。

## 核心要点

1. 三段式层级生成：先生成 HuBERT semantic token（保证语义连贯），再生成 SoundStream coarse token，最后生成 fine token
2. 首次证明离散 token + language model 可以生成高质量长时音频
3. 与 VALL-E 的关键区别：AudioLM 是 speech-to-speech（无文本条件），VALL-E 是 TTS（有文本条件）

## 代表工作

- AudioLM (Borsos et al., 2023): 原始论文
- [[VALL-E]]: 借鉴了层级建模思路，加入 text conditioning 变为 TTS

## 评测/常见数字

- LibriSpeech 续写：WER 6.0%（Whisper 评测）
- 无 speaker similarity 指标（speech continuation 设定）

## 相关概念

- [[Semantic Token]]
- [[Acoustic Token]]
- [[SoundStream]]
- [[HuBERT]]
- [[VALL-E]]
