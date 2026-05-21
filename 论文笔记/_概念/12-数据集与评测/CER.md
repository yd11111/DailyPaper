---
type: concept
aliases: [Character Error Rate, 字错率]
---

# CER

## 定义

**Character Error Rate（字错率）**：与 [[WER]] 同构，但以**字符**为基本单位。中文、日文、韩文等无空格分词语言中作为主指标，因为词边界模糊。

## 数学形式

$$
\text{CER} = \frac{S_c + D_c + I_c}{N_c}
$$

下标 $c$ 表示字符级。其他符号同 [[WER]]。

## 核心要点

1. 中文 ASR / TTS 评测主指标（[[AISHELL-1]]、[[WenetSpeech]]、[[Common Voice]] zh 等）。
2. 强模型在 [[AISHELL-1]] 上 CER ≈ 1%。
3. 与 WER 相比，对 OOV 词、新名词更不敏感。

## 代表工作

- 所有中文 ASR 论文（[[Paraformer]]、[[SenseVoice]]、[[Qwen3-ASR]]、[[MegaASR]]）

## 评测/常见数字

- [[AISHELL-1]]：强模型 0.7–1.5%
- [[WenetSpeech]] net：4–6%
- [[Common Voice]] zh：4–8%

## 相关概念

- [[WER]]
- [[ASR]]
