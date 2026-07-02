---
type: concept
aliases: [TF-GridNet, Time-Frequency GridNet]
---

# TF-GridNet

## 定义

Wang et al. (2023) 提出的时频域语音分离模型，使用交替的时间和频率维度 LSTM + 注意力机制在 T-F 网格上操作。在 WSJ0-2mix 等标准分离 benchmark 上达到 SOTA。

## 核心要点

1. 在时频两个维度上交替建模，捕获局部和全局依赖
2. 针对干净独白混合开发，在退化/对话条件下泛化有限
3. 与 Conv-TasNet 相比在分离质量上有显著提升，但计算量更大

## 相关概念

- [[Source Separation]]
- [[Conv-TasNet]]
- [[PIT]]
