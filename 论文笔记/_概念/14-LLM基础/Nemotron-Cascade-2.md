---
type: concept
aliases: [Nemotron-Cascade-2-30B-A3B, Nemotron Cascade 2]
---

# Nemotron-Cascade-2

## 定义

NVIDIA 的通用文本 LLM，30B 总参数 / 3B 激活参数的 Mixture-of-Experts 架构，采用 Mamba2-Transformer 混合设计。在推理/数学/代码/知识/对齐/长上下文/Agentic 任务上表现强劲，是 Audex 的文本骨架。

## 核心要点
1. 52 层，模型维度 2688，128 routable experts，6 activated
2. Mamba2-Transformer hybrid（兼具 Mamba 长序列效率和 Transformer 表达力）
3. 上下文长度 1M tokens
4. 原始词表 131,072
5. Yang et al., 2026

## 代表工作
- [[Audex]]: 以其 post-SFT checkpoint 为初始化，扩展音频能力

## 评测/常见数字
- AIME 2025: 92.4|98.6
- MMLU-Pro: 79.8
- NIAH@1M: 99.0

## 相关概念
- [[MoE]]
- [[Megatron-LM]]
- [[Audex]]
