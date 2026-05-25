---
type: concept
aliases: [SPSS, 统计参数语音合成, 参数合成]
---

# Statistical Parametric Speech Synthesis

## 定义
使用生成模型（HMM/DNN/RNN）预测声码器参数（频谱、F0、非周期性），再由声码器重建波形的 TTS 方法。与拼接合成相比，体积小且灵活，但自然度受限于三个退化因素。

## 核心要点
1. 流程：文本 → 语言学特征 → 声码器参数预测 → 声码器重建波形
2. 三大退化因素：声码器质量差（伪影）、生成模型精度不足、过平滑（闷声）
3. 两阶段训练次优：先拟合声码器提取参数、再建模参数轨迹
4. WaveNet 通过直接波形建模绕过了所有三个退化因素

## 代表工作
- [[WaveNet]]: 论文中作为对照的传统方法，WaveNet 将自然度差距缩小 51%~69%
- Zen et al. (2009): 统计参数合成的综述性工作

## 相关概念
- [[Linear Predictive Coding]]
- [[Mel-Spectrogram]]
