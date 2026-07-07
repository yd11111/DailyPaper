---
type: concept
aliases: [Scale-Invariant Signal-to-Noise Ratio, 尺度不变信噪比, SI-SDR]
---

# SI-SNR (Scale-Invariant SNR)

## 定义
尺度不变信噪比，衡量信号重建质量的指标。与传统 SNR 不同，SI-SNR 对目标信号的缩放不敏感，更适合评估语音分离和增强系统的性能。常作为训练损失使用。

## 数学形式

$$
\text{SI-SNR} = 10 \log_{10} \frac{\| s_{\text{target}} \|^2}{\| e_{\text{noise}} \|^2}
$$

其中 $s_{\text{target}} = \frac{\langle \hat{s}, s \rangle}{\|s\|^2} s$，$e_{\text{noise}} = \hat{s} - s_{\text{target}}$。

## 核心要点
1. 是语音分离（speech separation）领域的标准评测指标和训练目标
2. 与 SI-SDR（Scale-Invariant Signal-to-Distortion Ratio）数学上等价
3. 值越高越好，单位 dB
4. 在语音增强中常与频谱损失组合使用

## 代表工作
- [[Conv-TasNet]]: 使用 SI-SNR 作为训练目标
- [[LMPAN]]: 训练损失组合中包含 SI-SNR loss（权重 0.2）

## 相关概念
- [[PESQ]]
- [[STOI]]
- [[Source Separation]]
