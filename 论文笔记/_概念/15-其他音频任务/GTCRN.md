---
type: concept
aliases: [Gated Temporal Convolutional Recurrent Network]
---

# GTCRN

## 定义
Gated Temporal Convolutional Recurrent Network，一种用于语音增强的轻量级网络架构，结合时间卷积和循环网络。在 LMPAN 中用作核心增强分支，用 PConv（沿频率轴的 1x3 卷积）细化频谱特征。

## 核心要点
1. 设计目标是在低参数量下保持增强性能
2. 结合了 TCN 的并行计算优势和 RNN 的序列建模能力
3. 常用于端侧语音增强场景

## 代表工作
- Rong et al. (2024): GTCRN 原始论文
- [[LMPAN]]: 使用 GTCRN 作为增强分支

## 相关概念
- [[AEC]]
- [[Source Separation]]
