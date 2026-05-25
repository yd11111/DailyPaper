---
type: concept
aliases: [SDP, 随机时长预测器]
---

# Stochastic Duration Predictor

## 定义
VITS 引入的时长预测模块，基于 normalizing flow 将 duration 建模为随机变量而非确定值，使合成语音具有多样化的韵律和节奏。

## 核心要点
1. 训练时用 MAS 对齐得到的 duration 作为目标
2. 推理时从随机噪声采样 duration 并通过逆变换得到整数帧数
3. 比确定性 duration predictor（如 FastSpeech 2）生成更自然的韵律变化
4. 缺点：对某些说话人/句子可能产生不自然的极端 duration

## 代表工作
- [[VITS]]: 原始提出
- [[YourTTS]]: 继承使用，加入 speaker + language embedding 条件

## 相关概念
- [[Monotonic Alignment Search]]
- [[Grapheme-to-Phoneme]]
