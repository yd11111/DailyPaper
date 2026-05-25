---
type: concept
aliases: [空洞卷积, Atrous Convolution, Convolution with holes]
---

# Dilated Convolution

## 定义
一种在卷积核元素之间插入固定间隔（空洞）的卷积操作，使滤波器覆盖更大的输入区域而不增加参数量。最初用于信号处理中的小波变换，后广泛应用于图像分割和音频生成。

## 数学形式

$$
(F *_d k)(t) = \sum_{s} F(s) \cdot k(t - d \cdot s)
$$

- $d$: 空洞率（dilation factor）
- $d = 1$ 时退化为标准卷积

## 核心要点
1. 在不增加参数的情况下扩大感受野
2. 保持输入输出分辨率不变（无下采样）
3. 在信号处理（Holschneider 1989）和图像分割（Chen 2015, Yu & Koltun 2016）中有深厚历史
4. WaveNet 将其与因果性结合为 [[Dilated Causal Convolution]]

## 代表工作
- [[WaveNet]]: 空洞因果卷积用于音频波形生成
- DeepLab (Chen et al., 2015): 空洞卷积用于语义分割

## 相关概念
- [[Dilated Causal Convolution]]
- [[Causal Convolution]]
