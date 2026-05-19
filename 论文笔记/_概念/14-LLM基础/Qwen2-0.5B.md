---
type: concept
aliases: [Qwen2-0.5B, Qwen2 0.5B]
---

# Qwen2-0.5B

## 定义
阿里 2024 年发布的 Qwen2 系列最小参数模型（0.5B），decoder-only Transformer。

## 核心要点
1. GQA + RoPE + SwiGLU 标准架构
2. Tokenizer: BBPE, 词表 ~151k
3. 强中英双语能力，常作为研究 / 边端模型

## 代表工作
- [[OmniFlatten]] 用作 backbone，验证小模型 + 后训练即可做全双工对话

## 相关概念
- [[OmniFlatten]]
