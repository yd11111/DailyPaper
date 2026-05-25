---
type: concept
aliases: [ByteNet]
---

# ByteNet

## 定义
Kalchbrenner et al. (2017) 提出的用于序列到序列任务的神经网络架构，使用 [[Dilated Causal Convolution]] 替代 RNN，在机器翻译和字符级语言建模上表现出色。与 [[WaveNet]] 共享空洞卷积设计。

## 核心要点
1. 编码器使用空洞卷积（非因果），解码器使用 [[Dilated Causal Convolution]]（因果）
2. 时间复杂度为 $O(n)$，优于注意力机制的 $O(n^2)$
3. 与 WaveNet 共同推动了卷积序列模型的发展

## 相关概念
- [[Dilated Causal Convolution]]
- [[TCN]]
