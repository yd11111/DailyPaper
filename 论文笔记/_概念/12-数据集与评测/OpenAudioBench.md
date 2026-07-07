---
type: concept
aliases: [OAB, Open Audio Bench]
---

# OpenAudioBench

## 定义
语音问答（Speech QA）评测基准，包含 Llama Questions、Web Questions、TriviaQA 三个子集。支持 speech-to-text (s2t) 和 speech-to-speech (s2s) 两种评测模式。使用 LLM-as-judge（如 GPT-5.5）进行自动评分。

## 核心要点
1. 覆盖事实问答（TriviaQA）、常识问答（Llama Questions）、知识检索（Web Questions）三种类型
2. s2t 模式评估文本回答质量，s2s 模式额外评估语音生成质量
3. 常与 [[VoiceBench]] 组合使用，两者得分归一化至 0-100 后取平均作为"Overall"指标

## 代表工作
- [[FlexiSLM]]: 在此 benchmark 上报告了 7B SLM 最优分数
- Kimi-Audio-Evalkit: 将 OpenAudioBench 和 VoiceBench 整合为统一评测框架

## 相关概念
- [[VoiceBench]]
- [[LibriSpeech]]
