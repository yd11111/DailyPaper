---
type: concept
aliases: [Translatotron, Translatotron 2]
---

# Translatotron

## 定义
Google 提出的首个端到端语音到语音翻译（S2ST）模型。直接将源语言语音翻译为目标语言语音，跳过中间文本表示。Translatotron 2 进一步引入了说话人条件合成。

## 核心要点
1. Translatotron 1: 端到端架构，但难以保留源说话人的声音特征
2. Translatotron 2: 引入 speaker conditioning，但依赖 multi-speaker TTS 合成的伪双语数据
3. 质量仍不及级联系统（ASR→MT→TTS）
4. 后续被 SeamlessM4T、VALL-E X 等基于离散 token 的方法超越

## 代表工作
- [[VALL-E-X]]: 引用 Translatotron 作为 S2ST 领域先驱对比

## 相关概念
- [[S2ST]]
- [[Cross-Lingual TTS]]
