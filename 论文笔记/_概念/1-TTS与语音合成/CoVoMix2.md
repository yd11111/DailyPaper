---
type: concept
aliases: [CoVoMix v2]
---

# CoVoMix2

## 定义

2025 年微软提出的多说话人零样本对话生成模型，特点是**全 NAR + Flow Matching**——彻底放弃 AR，靠 mask-infill 一次性生成对话；适合短中等长度对话。

## 核心要点

1. 全 NAR Flow Matching，推理速度比 AR 快
2. 长序列稳定性受限——这是 [[VibeVoice]] 想要解决的痛点之一

## 代表工作

- 原论文 (arXiv 2506.00885, Zhang et al. 2025)
- [[VibeVoice]] 引言中作为"现有方法长度/稳定性不足"的代表

## 相关概念

- [[Flow Matching]]
- [[Mooncast]]
- [[VibeVoice]]
