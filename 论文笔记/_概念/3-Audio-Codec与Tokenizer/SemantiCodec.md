---
type: concept
aliases: [Semantic Codec]
---

# SemantiCodec

## 定义

把语义信息（来自 SSL audio encoder 或类似）整合到神经音频编解码器中的早期工作之一，输出 768 维 token，旨在让 codec token 既能重建音频又承载部分高层语义。在 [[LoSATok]] 中作为 "semantic-augmented codec" 路线的代表 baseline 出现。

## 核心要点

1. **混合 semantic + acoustic 表示**：早于 [[DashengTokenizer]] / [[LoSATok]] 等近期工作。
2. **768 维 token**：维度介于纯声学 codec (128 维 EnCodec) 和高维 unified tokenizer (1280 维 Dasheng) 之间。
3. **作为对比基线**：在 LoSATok Table 2 中报 XARES 平均 55.22 —— 优于纯声学 EnCodec (27.80) 但弱于现代 unified tokenizer。

## 代表工作

- 在 [[LoSATok]] 笔记中作为 XARES 理解 baseline 出现。

## 相关概念

- [[Mixed Token]]
- [[DashengTokenizer]] / [[LoSATok]]：路线后继。
- [[X-Codec]]：同期 semantic-acoustic 统一 codec。
