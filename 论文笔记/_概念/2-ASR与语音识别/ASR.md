---
type: concept
aliases: [Automatic Speech Recognition, 自动语音识别, 语音识别]
---

# ASR

## 定义

Automatic Speech Recognition：把语音信号转成文本的任务。现代主流分两路：CTC/RNN-T 端到端 (Conformer/Zipformer)、Encoder-Decoder Transformer ([[Whisper]]、Paraformer)。

## 核心要点

1. 输入：waveform 或 Mel；输出：字符 / BPE token / phoneme
2. 评测：WER (英)、CER (中)
3. **常被当作训练 Semantic Tokenizer 的代理任务**：让 codec encoder 输出特征对齐文本语义

## 代表工作

- [[Whisper]]
- [[Paraformer]]
- [[VibeVoice]]: 用 ASR 当 [[Semantic Tokenizer]] 训练的 proxy task

## 评测/常见数字

- LibriSpeech test-clean WER: ~2-3% (现代 SOTA)
- LibriSpeech test-other WER: ~4-6%

## 相关概念

- [[Whisper]]
- [[Paraformer]]
- [[Semantic Tokenizer]]
