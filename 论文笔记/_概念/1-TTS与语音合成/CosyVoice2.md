---
type: concept
aliases: [CosyVoice 2]
---

# CosyVoice2

## 定义

阿里通义实验室推出的 zero-shot TTS 系统第二版（Du et al., 2024b），基于 supervised semantic token + flow matching + vocoder 的级联架构。支持流式生成和情感指令控制。

## 核心要点

1. 使用有监督语义 token（区别于 HuBERT k-means 无监督 token）
2. 支持 instruct 模式（自然语言控制语速/情感/方言等）
3. 代码和模型已开源
4. 中文场景表现突出

## 评测/常见数字

- SeedTTS test-zh: SS 0.846, WER 1.451%
- 情感场景 ES: 0.802

## 代表工作

- Du et al. (2024b): CosyVoice 2 原始论文

## 相关概念

- [[CosyVoice]]
- [[Flow Matching]]
- [[Semantic Token]]
