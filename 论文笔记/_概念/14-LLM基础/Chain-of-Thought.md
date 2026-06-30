---
type: concept
aliases: [CoT, 思维链, Chain of Thought]
---

# Chain-of-Thought

## 定义
一种 LLM 推理范式：在给出最终答案前，让模型先显式生成中间推理步骤（reasoning trace），从而提升复杂推理任务的准确率。

## 核心要点
1. 由 Wei et al. (2022) 提出，在 few-shot prompt 中加入推理过程示例即可激活
2. Zero-shot CoT 变体只需加"Let's think step by step"
3. 后续发展：Tree-of-Thought、Self-Consistency、ReAct 等
4. 在语音领域应用：[[ModeratorLM]] 用 CoT 做多方轮转决策推理

## 代表工作
- Wei et al. 2022: 原始 CoT prompting 论文
- [[ModeratorLM]]: 将 CoT 应用于实时语音轮转决策

## 相关概念
- [[LoRA]]
- [[Turn-taking]]
