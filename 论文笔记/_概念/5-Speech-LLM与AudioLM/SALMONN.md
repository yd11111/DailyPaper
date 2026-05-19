---
type: concept
aliases: [SALMONN]
---

# SALMONN

## 定义
ICASSP 2024 提出的开源 speech/audio understanding LLM，speech-in / text-out。

## 核心要点
1. 双 encoder 设计：Whisper + BEATs 分别编码语音和音频事件
2. Q-Former 把 audio embedding 投到 LLM 空间

## 代表工作
- [[OmniFlatten]] 与之类比，强调 SALMONN 只能 understanding 不能生成 speech

## 相关概念
- [[OmniFlatten]]
