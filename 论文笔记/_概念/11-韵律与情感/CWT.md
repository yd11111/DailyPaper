---
type: concept
aliases: [Continuous Wavelet Transform, 连续小波变换, iCWT]
---

# CWT

## 定义

连续小波变换 (Continuous Wavelet Transform)，将时域信号分解为多尺度的时频表示。在 TTS 中主要用于将 F0 contour 分解为 pitch spectrogram，使 pitch predictor 能在多尺度上建模 pitch 变化。

## 数学形式

$$
W(\tau, t) = \tau^{-1/2} \int_{-\infty}^{+\infty} F_0(x) \psi\!\left(\frac{x - t}{\tau}\right) dx
$$

- 输入: F0 序列（帧级别）
- 输出: 多尺度 pitch spectrogram（通常 10 个尺度）
- 母小波: Mexican hat wavelet

## 核心要点

1. 比直接预测 F0 值更能捕捉多尺度的 pitch 变化模式
2. 可通过 iCWT 从预测的小波系数恢复 F0 contour
3. 在 FastSpeech 2 中离散化为 10 个尺度

## 代表工作

- [[FastSpeech2]]: 首次将 CWT 引入 TTS pitch 预测

## 相关概念

- [[Duration Predictor]]
- [[Phoneme]]
