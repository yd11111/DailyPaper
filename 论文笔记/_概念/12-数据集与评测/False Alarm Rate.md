---
type: concept
aliases: [误报率, 误警率, FPR, False Positive Rate, FA Rate]
---

# False Alarm Rate

## 定义
非目标样本中被模型错误判断为目标类别的比例，即 False Positive Rate。

## 数学形式

$$
\text{False Alarm Rate} = \frac{\text{FP}}{\text{FP} + \text{TN}}
$$

## 核心要点
1. 在 turn detection 中，高 FA rate 意味着系统不该打断时打断了用户
2. 与 Miss Rate 构成 trade-off：降低一个常导致另一个升高
3. 全双工对话中 FA 直接影响用户体验（被打断的挫败感）

## 代表工作
- [[FastTurn]]: turn detection 评测核心指标之一

## 相关概念
- [[Accuracy]]
- [[Miss Rate]]
