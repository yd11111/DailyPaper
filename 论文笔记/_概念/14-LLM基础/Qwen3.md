---
type: concept
aliases: [Qwen3 LLM, Qwen3 系列]
---

# Qwen3

## 定义
阿里通义千问团队发布的第三代大语言模型系列，提供多种参数规模（0.6B / 1.7B / 4B / 8B / 14B / 32B / 30B-A3B / 235B-A22B 等），支持 dense 和 MoE 架构，是 Qwen 多模态生态（Qwen3-Omni、Qwen3-TTS、Qwen3-ASR 等）的文本 backbone。

## 核心要点
1. 延续 Qwen2.5 架构，进一步扩大预训练数据规模和质量
2. 支持多语种文本处理，tokenizer 覆盖中英日韩等主要语种
3. 作为 Qwen3-TTS 和 Qwen3-Omni 等多模态模型的 LM backbone，提供文本理解和生成能力
4. 提供 ChatML 格式的对话模板，便于多模态任务统一训练

## 代表工作
- [[Qwen3-TTS]]: 使用 Qwen3 0.6B/1.7B 作为 TTS backbone
- [[Qwen3-Omni-30B-A3B]]: 使用 Qwen3 MoE 作为多模态 backbone

## 相关概念
- [[Qwen2.5]]
- [[Qwen-Audio]]
- [[Multi-Token Prediction]]
