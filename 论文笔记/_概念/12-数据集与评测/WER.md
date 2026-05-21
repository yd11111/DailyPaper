---
type: concept
aliases: [Word Error Rate, 词错率]
---

# WER

## 定义

**Word Error Rate（词错率）**：ASR / 语音识别 / TTS 可懂度的标准评测指标。基于编辑距离衡量预测词序列与参考词序列的差异。

## 数学形式

$$
\text{WER} = \frac{S + D + I}{N}
$$

- $S$：替换（substitution）词数
- $D$：删除（deletion）词数
- $I$：插入（insertion）词数
- $N$：参考文本总词数

## 核心要点

1. 越低越好，0 = 完美。
2. 计算前需要文本归一化：大小写、标点、数字、缩略词。
3. 在中文/日文等非空格分词语言里通常换用 [[CER]]（字符错率）。
4. TTS 客观可懂度评测：用 [[Whisper]] 等强 ASR 反解合成音频，算 WER。
5. 强 ASR 在干净集上 ~1%，鲁棒集（[[VOiCES]] / [[NOIZEUS]]）可能 10–30%，极端场景 >50%。

## 代表工作

- 所有 ASR 论文
- TTS 论文也用作可懂度指标（如 [[VALL-E]]、[[CosyVoice]]、[[F5-TTS]]）

## 评测/常见数字

- [[LibriSpeech]] test clean：强模型 1–2%
- [[VOiCES]]：强模型 7–12%
- [[NOIZEUS]] 0dB：强模型 20–40%
- TTS 可懂度（[[Seed-TTS-eval]]）：强模型 1–3%

## 相关概念

- [[CER]]
- [[ASR]]
- [[ASR Hallucination]]
- [[UTMOS]]
- [[SIM-O]]
