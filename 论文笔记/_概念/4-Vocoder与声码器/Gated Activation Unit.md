---
type: concept
aliases: [门控激活单元, Gated Activation, GLU]
---

# Gated Activation Unit

## 定义
一种非线性激活机制，将输入分为两路：一路经 $\tanh$ 得到候选值，一路经 $\sigma$ (sigmoid) 得到门控信号，二者逐元素相乘。源自 [[Gated PixelCNN]]，在音频建模中显著优于 [[ReLU]]。

## 数学形式

$$
\mathbf{z} = \tanh(W_f * \mathbf{x}) \odot \sigma(W_g * \mathbf{x})
$$

- $W_f$: filter 分支卷积核（控制"要表达什么"）
- $W_g$: gate 分支卷积核（控制"允许多少通过"）
- $\odot$: 逐元素乘法

## 核心要点
1. filter 分支提供候选激活，gate 分支调节信息流量
2. 在 WaveNet 初始实验中显著优于 ReLU
3. 概念上类似 LSTM 的门控机制，但用于卷积层
4. 后续在 Gated ConvNet (Dauphin et al., 2017) 中进一步发展

## 代表工作
- [[WaveNet]]: 音频波形建模中使用门控激活
- [[Gated PixelCNN]]: 图像生成中首次提出

## 相关概念
- [[Gated PixelCNN]]
- [[ReLU]]
