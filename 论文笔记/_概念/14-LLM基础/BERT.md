---
type: concept
aliases: [BERT, Bidirectional Encoder Representations from Transformers]
---

# BERT

## 定义
Google 2018 年提出的双向 Transformer 预训练语言模型，通过 Masked Language Modeling (MLM) 和 Next Sentence Prediction (NSP) 学习深层上下文文本表示。

## 核心要点
1. 双向编码：区别于 GPT 的单向，BERT 同时看左右上下文
2. 输出 token-level 和 sentence-level 表示，广泛用于下游任务
3. 在 TTS 中常提取上下文文本特征，提升韵律和停顿预测
4. Chinese-RoBERTa-WWM-Ext-Large 是中文 NLP 常用变体

## 代表工作
- [[GPT-SoVITS]]: 用 Chinese-RoBERTa 提取 1024 维文本特征，与 phoneme embedding 融合

## 相关概念
- [[Transformer]]
- [[G2P]]
