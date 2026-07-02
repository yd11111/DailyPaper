---
type: concept
aliases: [漏检率, 漏报率, FNR, False Negative Rate]
---

# Miss Rate

## 定义
目标类别中被模型遗漏（未检出）的样本比例，即 False Negative Rate。

## 数学形式

$$
\text{Miss Rate} = \frac{\text{FN}}{\text{TP} + \text{FN}}
$$

## 核心要点
1. 在 turn detection 中，高 miss rate 意味着系统该回复时没回复（漏判用户已说完）
2. 与 Recall 互补：Miss Rate = 1 - Recall
3. 低 miss rate 是全双工对话系统流畅度的关键

## 代表工作
- [[FastTurn]]: turn detection 评测核心指标之一

## 相关概念
- [[Accuracy]]
- [[False Alarm Rate]]
