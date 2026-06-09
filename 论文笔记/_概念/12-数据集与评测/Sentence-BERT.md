---
type: concept
aliases: [sBERT, SBERT, Sentence BERT]
---

# Sentence-BERT

## 定义
Sentence-BERT（sBERT）是基于 BERT 的句子嵌入模型，通过 Siamese 网络结构将句子映射到固定维度的向量空间，使得语义相似的句子具有高余弦相似度。常用作文本生成质量的语义评测指标。

## 核心要点
1. 将句子映射为固定维度的嵌入向量
2. 语义相似度通过余弦相似度计算，范围 [-1, 1]
3. 相比 BLEU 等 n-gram 指标，更能捕获语义层面的相似性

## 代表工作
- [[IRAF]]: 用 sBERT 语义相似度评估全双工对话响应质量

## 相关概念
- [[BERT]]
- [[BLEU]]
- [[余弦相似度]]
