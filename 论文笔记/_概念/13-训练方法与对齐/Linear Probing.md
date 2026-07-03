---
type: concept
aliases: [线性探测, Linear Probe, 线性分类器探测]
domain: LLM基础
tags: [interpretability, representation-analysis, probing]
created: 2026-07-03
---

# Linear Probing

## 定义

在预训练模型的中间层表示上训练一个简单线性分类器（通常是 logistic regression），以检测该层是否编码了特定属性（如情感、语义角色、语法结构等）。不更新模型参数，仅训练 probe 分类器。

## 核心要点

1. 线性 probe 精度越高，说明目标属性在该层的表示越线性可分——即模型在该层已"理解"该属性
2. 常用于可解释性研究，定位信息在模型哪些层被编码
3. 局限性：高 probe 精度不能完全排除非线性编码（线性 probe 可能是下界）

## 代表工作

- Alain & Bengio (2016): 提出用 linear classifier probes 理解中间层
- [[GeometricEmotionSteering]]: 用 linear probe 分析 CosyVoice2 SLM/CFM 的情感可分离度，对比 within-speaker vs cross-speaker 泛化

## 相关概念

- [[Activation Steering]]
- [[Local Intrinsic Dimensionality]]
