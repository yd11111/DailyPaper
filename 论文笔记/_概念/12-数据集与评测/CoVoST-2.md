---
type: concept
aliases: [CoVoST2, CoVoST-2, CoVoST 2]
domain: ASR
tags: [speech-translation, dataset, multilingual, s2st]
created: 2026-07-02
---

# CoVoST-2

## 定义
大规模多语种语音翻译数据集，基于 Common Voice 语音，覆盖 21 种语言到英语及 15 种语言间的翻译。常用于评估 S2T 和 S2S 翻译系统。

## 核心要点
1. 来源于 Mozilla Common Voice 语音数据，提供对应的翻译文本
2. 覆盖 X→EN（多语→英）方向的 21 种源语言
3. 是 S2ST（speech-to-speech translation）评测中常用的标准 benchmark
4. CVSS 子集提供合成的目标端语音，方便 S2S 训练

## 代表工作
- [[PRIME-Speech]]: 使用 CoVoST-2 X2EN 1k h 作为训练数据 + FLEURS/CoVoST-2 作为评测
- [[Qwen2.5-Omni]]: 在 CoVoST-2 上评测翻译 S2S 能力

## 评测/常见数字
- CoVoST-2 X2EN BLEU: 强系统 35-42（S2T），S2S 通常低 3-10 分
- 七种常用源语言（de/fr/es/ca/it/ru/zh）是主要评测配置

## 相关概念
- [[LibriSpeech]]
- [[FLEURS]]
