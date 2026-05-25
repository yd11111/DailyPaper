---
type: concept
aliases: [Mel-Cepstral Distortion, 梅尔倒谱失真]
---

# MCD

## 定义

Mel-Cepstral Distortion，衡量合成语音与参考语音在梅尔倒谱域的距离。越低越好。常用于评估韵律/情感保持能力。

## 数学形式

$$
\text{MCD} = \frac{10\sqrt{2}}{\ln 10} \sqrt{\sum_{i=1}^{D} (c_i^{\text{ref}} - c_i^{\text{syn}})^2}
$$

- $c_i^{\text{ref}}, c_i^{\text{syn}}$: 参考/合成语音的第 $i$ 维梅尔倒谱系数（MCEP）
- $D$: 梅尔倒谱维度（通常 13 或 24）
- 单位: dB

## 核心要点
1. 主要用于韵律/情感 TTS 的评测（如 RAVDESS 数据集）
2. 越低表示合成语音与参考在韵律上越接近
3. MCD-Acc: 基于 MCD 特征做情感分类的准确率，衡量情感保持能力

## 代表工作
- [[NaturalSpeech3]]: 用 MCD 和 MCD-Acc 评估 RAVDESS 上的情感韵律保持

## 相关概念
- [[Mel-Spectrogram]]
- [[PESQ]]
- [[UTMOS]]
