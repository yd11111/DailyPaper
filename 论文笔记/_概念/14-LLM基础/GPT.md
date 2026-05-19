---
type: concept
aliases: [GPT, Decoder-only Transformer, Causal LM]
---

# GPT

## 定义
Generative Pretrained Transformer，单向 (causal) 自回归 decoder-only Transformer 范式。

## 核心要点
1. Next-token prediction 训练目标
2. 因果 mask 保证只看左侧上下文，天然支持流式生成
3. OmniFlatten 等工作核心 "无架构改动" 的对象

## 代表工作
- 几乎所有现代 LLM；[[OmniFlatten]] 复用其原生 AR 解码处理 flatten 后的多模态序列

## 相关概念
- [[OmniFlatten]]
