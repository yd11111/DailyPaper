---
type: concept
aliases: [Focus Rate, 聚焦率]
---

# Focus Rate

## 定义
衡量 encoder-decoder attention 矩阵对角性程度的指标，用于在多头注意力中选择最接近单调对齐的 attention head。

## 数学形式

$$
F = \frac{1}{S} \sum_{s=1}^{S} \max_{1 \leq t \leq T} a_{s,t}
$$

- $S$: mel-spectrogram 长度
- $T$: phoneme 序列长度
- $a_{s,t}$: attention 权重矩阵元素

## 核心要点
1. Focus Rate 越高，attention 越集中（越接近硬对齐/对角线）
2. FastSpeech 中用于从教师模型的多个 attention head 中选择最优 head 提取时长
3. 完美对角 attention 的 Focus Rate = 1.0

## 代表工作
- [[FastSpeech]]: 首次提出 Focus Rate 概念用于 attention head 选择

## 相关概念
- [[Attention Alignment]]
- [[Duration Predictor]]
- [[Forced Alignment]]
