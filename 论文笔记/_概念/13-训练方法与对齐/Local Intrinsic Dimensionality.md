---
type: concept
aliases: [LID, 局部内在维度, 局部固有维度]
domain: LLM基础
tags: [manifold-learning, geometric-analysis, representation-analysis]
created: 2026-07-03
---

# Local Intrinsic Dimensionality (LID)

## 定义

量化数据点在其局部邻域内的流形维度——即该点附近数据分布的有效自由度数。高 LID 表示局部几何复杂、约束少；低 LID 表示局部结构紧凑、高度约束。

## 数学形式

常用 Levina-Bickel MLE 估计器（Levina & Bickel, 2004）：基于样本点到 K 个最近邻的距离排序 $r_1, r_2, \ldots, r_K$ 估计局部维度。

关键派生指标 $\Delta\text{LID}$：

$$
\Delta\text{LID} = \text{LID}_{\text{pooled}} - \overline{\text{LID}}_{\text{per-class}}
$$

- $\Delta\text{LID} > 0$: 不同类别贡献了独立的变化方向 → 适合组合控制
- $\Delta\text{LID} < 0$: 类别共享同一流形 → 控制方向互相纠缠

## 核心要点

1. LID 是局部度量（per-sample），可以揭示全局维度统计无法捕捉的流形结构
2. 在 NLP/vision 中用于分析 transformer 的表示动力学（Valeriani et al., 2023）
3. 参数 K（近邻数）的选择影响估计精度——过小则噪声大，过大则平滑掉局部结构

## 代表工作

- Levina & Bickel (2004): 提出 MLE 估计方法
- Amsaleg et al. (2015): 将 LID 用于数据分析
- Valeriani et al. (2023): 分析大型 transformer 的隐表示几何
- [[GeometricEmotionSteering]]: 用 ΔLID 评估 TTS SLM/CFM 情感方向的可组合性

## 相关概念

- [[Linear Probing]]
- [[Activation Steering]]
