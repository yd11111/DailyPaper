---
type: concept
aliases: [emotion2vec, Emotion2Vec]
---

# emotion2vec

## 定义

开源语音情感表示模型（Ma et al., 2024），从语音中提取情感特征向量。常用于计算情感相似度（Emotion Similarity, ES）评测指标。

## 核心要点

1. 自监督或有监督预训练的语音情感表示学习
2. 输出的情感嵌入可用于计算余弦相似度（ES 指标）
3. 开源可用，被多个 TTS 情感评测采用

## 代表工作

- [[IndexTTS2]]: 使用 emotion2vec 计算 ES 评测指标

## 相关概念

- [[ESD]]
- [[Conformer]]
