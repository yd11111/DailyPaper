---
type: concept
aliases: []
---

# DualCodec

## 定义

双码本语音 tokenizer，将语音同时编码为 semantic token 和 acoustic token 两路离散表示。与 [[SpeechTokenizer]] 的 RVQ 隐式分层不同，DualCodec 使用独立的编码路径实现语义-声学分离。

## 核心要点

1. 双路编码：semantic path 保留语言内容，acoustic path 保留音色/风格
2. 目标是为 Speech LLM 提供更高效的 token 接口（LLM 只建模 semantic，decoder 补 acoustic）
3. [[DSA-Tokenizer]] 将其作为对比 baseline，声称通过 ASR/mel 显式约束实现更好的解耦

## 代表工作

- DualCodec (2024)
