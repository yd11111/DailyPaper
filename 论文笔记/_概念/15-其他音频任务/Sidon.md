---
type: concept
aliases: [Sidon]
---

# Sidon

## 定义

Nakata et al. (2026a) 提出的语音恢复模型，在 SSL 模型（w2v-BERT 2.0）的隐空间操作，将退化语音的 SSL 特征映射到干净语音的 SSL 特征，再通过声码器重建波形。DialogueSidon 的前作和基础。

## 核心要点

1. 利用 SSL 模型对退化信号的天然鲁棒性，在 SSL 特征空间做恢复而非波形空间
2. 使用 LoRA 适配 w2v-BERT 2.0 以适应退化输入
3. 被用于清洗 Fisher、CALLHOME 等电话录音数据集，为下游系统提供更干净的训练目标
4. 仅处理单说话人，不做分离——[[DialogueSidon]] 将其扩展到两说话人对话场景

## 代表工作

- Nakata et al. 2026a: 原始 Sidon 论文
- [[DialogueSidon]]: 对话扩展版

## 相关概念

- [[Speech Restoration]]
- [[w2v-BERT 2.0]]
- [[LoRA]]
- [[DialogueSidon]]
