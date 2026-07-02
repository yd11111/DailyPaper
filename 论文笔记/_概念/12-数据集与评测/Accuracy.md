---
type: concept
aliases: [准确率, 分类准确率]
---

# Accuracy

## 定义
分类任务中正确预测样本占总样本的比例，衡量模型整体分类正确率。

## 数学形式

$$
\text{Accuracy} = \frac{\text{TP} + \text{TN}}{\text{TP} + \text{TN} + \text{FP} + \text{FN}}
$$

## 核心要点
1. 最基本的分类评测指标
2. 类别不均衡时可能产生误导（如 99% 负样本时全预测负也有 99% accuracy）
3. 常与 Precision / Recall / F1 / Miss Rate / False Alarm Rate 配合使用

## 代表工作
- [[FastTurn]]: 用于 turn detection 评测

## 相关概念
- [[Miss Rate]]
- [[False Alarm Rate]]
