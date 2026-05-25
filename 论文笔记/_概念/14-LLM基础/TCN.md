---
type: concept
aliases: [Temporal Convolutional Network, 时序卷积网络]
---

# TCN

## 定义
基于因果卷积和空洞卷积的序列建模架构，由 Bai et al. (2018) 系统总结。核心思想来源于 [[WaveNet]] 的 [[Dilated Causal Convolution]]，在多种序列任务上与 RNN 竞争。

## 核心要点
1. 使用 [[Causal Convolution]] + [[Dilated Convolution]] 处理序列
2. 训练可并行化，推理效率高于 RNN
3. 在音频、时间序列预测等领域广泛应用

## 代表工作
- [[WaveNet]]: TCN 架构的先驱
- Bai et al. (2018): "An Empirical Evaluation of Generic Convolutional and Recurrent Networks for Sequence Modeling"

## 相关概念
- [[Dilated Causal Convolution]]
- [[Causal Convolution]]
