---
type: concept
aliases: [Miipher, Miipher-2]
---

# Miipher

## 定义

Koizumi et al. (2023) 提出的基于 SSL 特征的语音恢复模型。利用 w2v-BERT 等 SSL 模型提取退化输入的鲁棒特征，预测干净语音的 Mel 频谱后用声码器合成。用于创建 FLEURS-R 数据集。

## 核心要点

1. 核心思路：SSL 模型对退化信号天然鲁棒 → 在 SSL 特征空间做映射而非波形空间
2. 用于大规模数据集清洗（FLEURS-R），证明了语音恢复在 TTS 训练数据 pipeline 中的价值
3. Miipher-2 (Karita et al. 2025) 是改进版
4. 与 [[Sidon]] 思路相近，但 Sidon 进一步在 SSL 隐空间引入 VAE + LoRA 适配

## 相关概念

- [[Speech Restoration]]
- [[w2v-BERT 2.0]]
- [[Sidon]]
