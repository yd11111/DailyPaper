---
type: concept
aliases: [Spark-TTS, Spark TTS]
---

# SparkTTS

## 定义

基于单一 LLM 的 zero-shot TTS 系统（Wang et al., 2025），使用 BiCodec 进行音频编解码，将 TTS 完全建模为语言模型任务。

## 核心要点

1. 单一 LLM 架构（非级联），简化系统设计
2. 使用 BiCodec 实现音频离散化
3. 支持通过属性标签控制时长等属性

## 评测/常见数字

- SeedTTS test-en: SS 0.755, WER 1.543%
- 在多个基准上 SS 偏低，可能因 BiCodec 编码信息有限

## 代表工作

- Wang et al. (2025): Spark-TTS 原始论文

## 相关概念

- [[Autoregressive]]
- [[Semantic Token]]
