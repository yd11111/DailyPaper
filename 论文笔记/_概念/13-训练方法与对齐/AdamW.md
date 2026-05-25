---
type: concept
aliases: [Adam with Weight Decay, AdamW optimizer]
---

# AdamW

## 定义
Adam 优化器的改进版本，将权重衰减（weight decay）从梯度更新中解耦出来，直接作用于参数本身，而非嵌入 L2 正则化项。

## 核心要点
1. 修正了 Adam 中 L2 正则化与自适应学习率交互的问题
2. 在大多数深度学习任务中表现优于标准 Adam
3. 广泛用于 Transformer、TTS、ASR 等模型训练

## 代表工作
- [[YourTTS]]: $\beta_1=0.8, \beta_2=0.99$, weight decay=0.01
- [[VITS]]: 端到端 TTS 训练

## 相关概念
- [[Transformer]]
