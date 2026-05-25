---
type: concept
aliases: [NaturalSpeech3, NS3]
---

# NaturalSpeech 3

## 定义

微软 2024 年提出的零样本 TTS 模型，采用 factorized codec + diffusion 两阶段方案：先将语音分解为内容、韵律、声学细节等子空间的离散/连续表示，再用 diffusion 模型逐子空间生成。500M 参数，60K 小时英文数据训练。

## 数学形式

将语音表示分解为多个子空间：
$$
x = f(z_{\text{content}}, z_{\text{prosody}}, z_{\text{timbre}}, z_{\text{detail}})
$$

每个子空间独立建模后组合。

## 核心要点

1. Factorized codec 将语音分解为多粒度子空间，分别建模
2. 500M 参数，60K 小时英文训练
3. 在 LibriSpeech 40 样本子集上 WER 1.94%、SIM 0.67

## 代表工作

- NaturalSpeech 3 原论文 (Ju et al., Microsoft, 2024)
- [[F5-TTS]]: 作为对比基线之一

## 评测/常见数字

- LibriSpeech 40-sample WER: 1.94%, SIM: 0.67, RTF: 0.296

## 相关概念

- [[Flow Matching]]
- [[VALL-E]]
- [[Voicebox]]
- [[Duration Predictor]]
