---
type: concept
aliases: [Speech-to-Speech Translation, 语音到语音翻译]
---

# S2ST

## 定义
Speech-to-Speech Translation，语音到语音翻译。直接将源语言语音翻译为目标语言语音，理想情况下保留说话人音色和韵律。

## 核心要点
1. 传统方法为级联系统：ASR → MT → TTS，存在误差累积和信息丢失
2. 端到端 S2ST 试图用单一模型直接完成语音到语音的翻译
3. 保留说话人身份（voice preservation）是关键挑战
4. 近年来离散 token 方法（如 VALL-E X）展示了零样本 voice-preserving S2ST 的可能

## 代表工作
- [[Translatotron]]: Google 提出的首个端到端 S2ST 模型
- [[VALL-E-X]]: 微软基于 codec language model 的零样本 S2ST
- [[SeamlessM4T]]: Meta 的多模态多语种翻译系统

## 相关概念
- [[Cross-Lingual TTS]]
- [[ASR]]
