---
type: concept
aliases: [语义 tokenizer]
---

# Semantic Tokenizer

## 定义

只承载"说什么"（语言内容）的 speech tokenizer。通常用 [[ASR]] 或 [[HuBERT]] / [[WavLM]] 等 SSL 特征做 k-means 聚类得到，对音色不敏感。与 [[Acoustic Tokenizer]] 配合使用是 AudioLM / SpeechTokenizer 的标准分层。

## 核心要点

1. 训练目标：让特征对齐文本语义（ASR proxy / SSL k-means）
2. 帧率较低 (50 Hz 量级)
3. **VibeVoice 用法**：与 acoustic latent 拼成 hybrid 表示送 LLM，提升内容理解

## 代表工作

- AudioLM (Borsos et al. 2023)
- [[VibeVoice]]: 使用 ASR proxy 训练的 semantic encoder（架构镜像 acoustic tokenizer 但去掉 VAE）

## 相关概念

- [[Acoustic Tokenizer]]
- [[ASR]]
- [[HuBERT]]
- [[WavLM]]
