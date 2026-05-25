---
type: concept
aliases: [Transformer 架构]
---

# Transformer

## 定义
基于自注意力机制的序列建模架构（Vaswani et al., 2017），是现代 LLM、TTS LLM、ASR 等几乎所有序列模型的基础 backbone。

## 核心要点
1. Self-Attention 机制实现全局依赖建模
2. CosyVoice 的 Text Encoder 和 LLM 都是 Transformer 架构
3. 变体：Conformer（ASR）、DiT（Diffusion）

## 代表工作
- [[CosyVoice]]: Text Encoder + LLM 均为 Transformer
- [[Conformer]]: Transformer + CNN 混合

## 相关概念
- [[Conformer]]
