---
type: concept
aliases: [STFT, Short-Time Fourier Transform, 短时傅里叶变换, iSTFT]
---

# STFT

## 定义
短时傅里叶变换，将时域波形通过滑动窗口分帧后做 FFT，得到时频表示。逆变换 iSTFT 可从频谱恢复波形。是音频处理的基础运算。

## 数学形式

$$
X(m, k) = \sum_{n=0}^{N-1} x(n + mH) \cdot w(n) \cdot e^{-j2\pi kn/N}
$$

- $m$: 帧索引, $k$: 频率 bin, $N$: FFT size, $H$: hop size, $w(n)$: 窗函数

## 核心要点
1. 是 [[Mel-Spectrogram]] 计算的前置步骤（STFT → Mel 滤波器组）
2. 典型参数：FFT size 1024, window 1024, hop 256（VITS 设定）
3. iSTFT 是部分现代声码器的核心组件（如 iSTFT-Net、[[Vocos]]）

## 代表工作
- [[VITS]]: STFT 用于 mel 重建损失计算
- [[Vocos]]: 直接预测 STFT 系数再 iSTFT 合成

## 相关概念
- [[Mel-Spectrogram]]
- [[Vocoder]]
