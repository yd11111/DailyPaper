---
type: concept
aliases: [MTP, Multi-Token Prediction, 多token预测, Speculative Decoding]
---

# Multi-Token Prediction

## 定义

在自回归解码中，每一步前向不仅预测下一个 token，还通过多个辅助分支并行预测未来 $H$ 个 token 的技术。配合验证步骤可在不损失准确性的前提下大幅加速推理。

## 核心要点

1. 训练时多个辅助 head 分别预测 $x_{t+2}, \ldots, x_{t+H+1}$，损失按指数衰减加权。
2. 推理时产生 $(H+1)$ 个 token 提案，通过自回归验证逐个检查——一旦某个 future token 不一致，后续全部拒绝。
3. 在 grounded generation（如 ASR）中特别有效，因为外部模态约束了输出分布，接受率高。
4. [[Step-Audio-2.5]] 使用 MTP-5（5 辅助分支），平均接受长度 5.0/6，RTF 0.0053。

## 代表工作

- Meta, 2024: "Better & Faster Large Language Models via Multi-token Prediction"
- StepAudio 2.5 (StepFun, 2026): MTP-5 用于 ASR 加速

## 相关概念

- [[Speculative Decoding]]
- [[Step-Audio-2.5]]
- [[ASR]]
