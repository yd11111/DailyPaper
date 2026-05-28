---
type: concept
aliases: [Q-Former, QFormer, Querying Transformer]
---

# Q-Former

## 定义
Querying Transformer，源自 BLIP-2（Li et al. 2023），通过 learnable query tokens 与 cross-attention 从冻结的单模态编码器中提取固定数量的条件 token，作为 LLM 的前缀输入。本质是一种模态桥接模块。

## 核心要点
1. 由 learnable query vectors + cross-attention layers 组成
2. 原用于视觉-语言对齐（从冻结 ViT 提取视觉 token 给 LLM）
3. PilotTTS 将其迁移到语音领域：从冻结 w2v-BERT 2.0 提取风格条件 token（32 个）
4. 优势：输出长度固定、与输入长度解耦、鲁棒性强

## 代表工作
- BLIP-2: 首次提出 Q-Former 架构用于视觉-语言对齐
- [[PilotTTS]]: 将 Q-Former 迁移到语音 TTS，用于说话人风格条件提取

## 相关概念
- [[PerceiverResampler]]
- [[Cross-Attention]]
- [[In-Context Learning]]
