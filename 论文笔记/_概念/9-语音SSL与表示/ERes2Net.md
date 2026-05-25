---
type: concept
aliases: [Enhanced Res2Net, ERes2Net Speaker Encoder]
---

# ERes2Net

## 定义

增强版 Res2Net 说话人验证模型，通过局部和全局特征融合实现高精度说话人 embedding 提取，常用于 TTS 零样本评测中的说话人相似度计算。

## 核心要点

1. 基于 Res2Net 架构，增加局部和全局特征融合模块
2. 输出固定维度的说话人 embedding，用余弦相似度衡量说话人一致性
3. 与 WavLM-based speaker encoder 互为补充（CosyVoice 3 同时报告两者结果）

## 代表工作

- Chen et al. 2023: "An enhanced Res2Net with local and global feature fusion for speaker verification"
- [[CosyVoice3]]: 用 ERes2Net 作为说话人相似度评测指标之一

## 相关概念

- [[WavLM]]
- [[Seed-TTS-eval]]
