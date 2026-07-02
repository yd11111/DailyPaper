---
type: concept
aliases: [ConvTasNet, Convolutional Time-domain Audio Separation Network]
---

# Conv-TasNet

## 定义

Luo & Mesgarani (2019) 提出的端到端时域语音分离模型，使用 1-D 卷积编码器-解码器框架替代 STFT，通过时间卷积网络 (TCN) 学习分离掩码。是语音分离领域的里程碑式基线。

## 核心要点

1. 完全在时域操作，避免了频域方法的相位估计问题
2. 编码器-分离-解码器三段架构，分离网络使用堆叠的膨胀 1-D 卷积
3. 使用 [[PIT]] 解决排列歧义
4. 参数量小（~5M），推理快，但在高度重叠和退化条件下性能有限

## 代表工作

- Luo & Mesgarani 2019: 原始论文
- 后续改进：SepFormer、TF-GridNet 等在结构上有大幅提升

## 相关概念

- [[PIT]]
- [[Source Separation]]
- [[TF-GridNet]]
