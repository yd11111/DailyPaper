---
type: concept
aliases: [MinMo LLM, 多模态语音理解模型]
---

# MinMo

## 定义

阿里通义实验室开发的大规模多模态语音理解模型，在超过 140 万小时语音数据上预训练，支持 ASR、语言识别、情感识别、音频事件检测、说话人分析等多种语音理解任务。

## 核心要点

1. 架构包含 Voice Encoder（Transformer + RoPE）+ LLM，支持语音输入多任务理解
2. 预训练数据规模达 140 万小时，覆盖多语种
3. 作为 CosyVoice 3 speech tokenizer 的基座模型，在其 Voice Encoder 中间插入 FSQ 模块做 token 提取
4. 相比 SenseVoice-Large（仅 ASR 训练），MinMo 的多任务预训练使 token 编码更丰富的副语言信息

## 代表工作

- Chen et al. 2025: "MinMo: A multimodal large language model for seamless voice interaction"
- [[CosyVoice3]]: 基于 MinMo 构建监督多任务 speech tokenizer

## 相关概念

- [[SenseVoice]]
- [[Speech LLM]]
- [[FSQ]]
- [[Speech Tokenizer]]
