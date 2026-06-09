---
type: concept
aliases: [BLEU Score, Bilingual Evaluation Understudy]
---

# BLEU

## 定义
BLEU（Bilingual Evaluation Understudy）是一种自动评测指标，通过计算候选文本与参考文本之间的 n-gram 精确率来衡量文本生成质量。原设计用于机器翻译，后广泛用于各类文本生成任务。

## 数学形式

$$
\text{BLEU} = \text{BP} \cdot \exp\left(\sum_{n=1}^{N} w_n \log p_n\right)
$$

- $p_n$: n-gram 精确率
- $\text{BP}$: 简短惩罚（Brevity Penalty）
- $w_n$: 各 n-gram 的权重（通常均匀 $1/N$）

## 核心要点
1. 范围 0-100（或 0-1），越高越好
2. 主要衡量词汇级别的精确匹配，不直接捕获语义相似性
3. 在语音对话系统中常用于评估 ASR 转写后的响应质量

## 代表工作
- [[IRAF]]: 用 BLEU 评估全双工对话系统的响应质量

## 相关概念
- [[WER]]
- [[Sentence-BERT]]
