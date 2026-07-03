---
type: concept
aliases: [Consistency Models, CM, 一致性模型]
---

# Consistency Model

## 定义
Consistency Model 是 Song et al. (2023) 提出的生成模型加速方法，核心思想是训练模型使得 ODE 轨迹上任意一点都能直接映射到轨迹终点（即生成结果），从而支持单步或少步生成。与 [[Rectified Flow]] 的路径矫直思路互补。

## 核心要点
1. 自一致性约束：ODE 轨迹上任意两点经过模型映射后得到相同终点
2. 支持单步生成（极致加速）或多步精化（质量-速度权衡）
3. 可以从预训练的扩散模型蒸馏（Consistency Distillation）或直接训练（Consistency Training）
4. 在图像生成中广泛验证，在语音领域也有应用探索

## 代表工作
- Song et al., 2023: 原始论文
- [[UnifiedGuidanceFM]]: 作为相关加速方法被引用对比

## 相关概念
- [[Rectified Flow]]
- [[Flow Matching]]
- [[NFE]]
- [[Knowledge Distillation]]
