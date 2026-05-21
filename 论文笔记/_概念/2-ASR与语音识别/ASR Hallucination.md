---
type: concept
aliases: [ASR 幻觉, Speech Hallucination, 语音识别幻觉]
---

# ASR Hallucination

## 定义

ASR / [[Speech LLM]] 在输入音频质量差（重噪声、远场、静默、超出域）时，输出**与音频内容无关的虚构文本**的现象。常表现为：复读、重复模板句、整段编造、用训练分布里高频短语填充。

## 核心要点

1. 是大模型化 [[ASR]]（[[Speech LLM]] / Omni 模型）的典型副作用。背后机制是 LLM 在声学锚定丢失时回退到语言先验。
2. 与 **漏识（omission）** 是两种主要错误模式：低 [[WER]] 段以替换/删除为主，高 WER 段以幻觉/漏识为主。
3. [[Whisper]] 在长静默、纯噪声段尤其严重（典型场景：整段输出 "Thanks for watching"）。
4. [[MegaASR]] 用 [[Anti-Repetition Reward]] 硬门控 + [[Sentence-Level Reconstruction Reward]] 抑制；LLM-as-judge 评估上 Hall. 18.7 → 11.8。

## 评测方式

- LLM-as-judge 给出 Hallucination / Missed / Semantic / Key Entity 四维评分
- 人工标注高 [[WER]] 段的输出类型分布

## 代表工作

- [[MegaASR]]：系统化抑制 ASR 幻觉
- 各种 [[Whisper]] 改进工作

## 相关概念

- [[WER]]
- [[ASR]]
- [[Speech LLM]]
- [[Anti-Repetition Reward]]
