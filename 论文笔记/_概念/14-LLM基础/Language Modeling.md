---
type: concept
aliases: [语言建模, LM, Causal Language Modeling, CLM]
---

# Language Modeling

## 定义

给定上文预测下一个 token 的任务范式。在因果语言建模（Causal LM）中，模型以自回归方式逐 token 生成，每个 token 的预测仅依赖于它之前的所有 token。

## 数学形式

$$
p(\mathbf{x}) = \prod_{t=1}^{T} p(x_t \mid x_1, \ldots, x_{t-1})
$$

## 核心要点
1. GPT 系列、LLaMA 等 decoder-only 模型的基础训练范式
2. 在 TTS 领域被用于将语音合成重构为条件语言建模问题（给定文本+参考音频 token → 预测目标语音 token）
3. Teacher forcing 训练 + 自回归推理的标准模式

## 代表工作
- [[VALL-E]]: 将 TTS 当作条件语言建模问题
- [[Sarashina22-TTS]]: 以日语 LLM 为 backbone 的 TTS 语言建模
- [[Qwen3-TTS]]: LLM-native TTS

## 相关概念
- [[Cross-Entropy Loss]]
- [[Autoregressive Model]]
- [[Semantic Token]]
