---
type: concept
aliases: [Generative Spoken Language Model, dGSLM]
---

# GSLM

## 定义

Meta 提出的 Generative Spoken Language Model，将语音用 HuBERT/CPC 等 SSL 模型离散化后，用 language model 建模并生成语音。是纯语音（无文本）的生成式语言模型。dGSLM 是其双说话人对话扩展版本。

## 核心要点

1. 纯语音到语音的生成式模型，不依赖文本转录
2. 使用 SSL 特征的 k-means 聚类作为离散 token
3. 缺点：丢失说话人身份信息（speaker similarity 极低，仅 0.126）
4. dGSLM 扩展到双说话人对话建模，是全双工对话建模的早期探索

## 代表工作

- GSLM (Lakhotia et al., 2021): 原始单说话人版本
- dGSLM (Nguyen et al., 2023): 双说话人对话扩展
- [[VALL-E]]: 改用 codec token 保留说话人信息，加入 text conditioning

## 评测/常见数字

- LibriSpeech WER 12.4%, SPK similarity 0.126（远低于 VALL-E 的 0.580）

## 相关概念

- [[HuBERT]]
- [[Semantic Token]]
- [[AudioLM]]
- [[VALL-E]]
- [[Full-Duplex]]
