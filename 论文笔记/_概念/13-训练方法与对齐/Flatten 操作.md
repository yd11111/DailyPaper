---
type: concept
aliases: [Flatten 操作, Flatten Operation, Flatten]
---

# Flatten 操作

## 定义
把多个并行模态/任务流（如 user-speech / user-text / assistant-text / assistant-speech）按 chunk 顺序串联成单条序列的数据组织技巧。

## 核心要点
1. 目的：让原生 [[GPT]] 自回归直接处理多流多任务，无需结构改动
2. Chunk size 是关键超参（OmniFlatten: speech=10, text=2 token）
3. Chunk 内顺序：input speech → output text → output speech
4. 用 silent_speech_token / silent_text_token 填空保持流对齐
5. 对比 [[Moshi]] 多流并行（需 acoustic delay + inner monologue）

## 代表工作
- [[OmniFlatten]] 提出，是其核心创新

## 相关概念
- [[OmniFlatten]]
