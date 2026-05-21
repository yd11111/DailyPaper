---
type: concept
aliases: [Afri HuBERT, African HuBERT]
---

# AfriHuBERT

## 定义

非洲多语种语音的 [[HuBERT]] 微调版本（社区项目），把 HuBERT 自监督预训练扩展到约鲁巴、豪萨、伊博、斯瓦希里等多种非洲语言；常用作下游 ASR / LID 的特征编码器。

## 核心要点

1. **继承 HuBERT 自监督训练范式**：迭代 k-means clustering → mask prediction
2. **预训练语料**：非洲多语种语音库（NaijaVoices、AfriSpeech 等的子集）
3. **目的**：缓解通用 SSL 模型（HuBERT / wav2vec 2.0 / XLS-R）在非洲语种下游任务的迁移损失
4. 在 LID 任务上替代 [[VoxLingua107]] 一类基础 LID 是常见做法

## 代表工作

- [[SBPN]]：尼日利亚多语种 ASR 蒸馏 pipeline 中作为 backbone 之一

## 评测/常见数字

- 在 AfriSpeech 5 语种 LID 上准确率 > 90%

## 相关概念

- [[HuBERT]]
- [[wav2vec 2.0]]
- [[VoxLingua107]]
