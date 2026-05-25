---
type: concept
aliases: [Attentron]
---

# Attentron

## 定义
基于注意力机制的少样本/零样本 TTS 系统，使用细粒度编码器（fine-grained encoder）从多个参考样本中提取说话人风格，并结合粗粒度编码器（coarse-grained encoder）实现更好的音色迁移。

## 核心要点
1. 使用 attention 从多个参考音频中提取可变长度的风格 embedding
2. 细粒度+粗粒度双编码器设计
3. 在 VCTK 上的零样本 TTS baseline：SECS 0.731, MOS 3.86

## 代表工作
- [[YourTTS]]: 作为 ZS-TTS baseline 被对比

## 评测/常见数字
- VCTK ZS-TTS: SECS 0.731, MOS 3.86, Sim-MOS 3.30

## 相关概念
- [[Speaker Encoder]]
- [[VITS]]
