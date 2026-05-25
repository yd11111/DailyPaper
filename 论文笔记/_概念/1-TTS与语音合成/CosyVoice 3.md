---
type: concept
aliases: [CosyVoice3, CosyVoice-3]
---

# CosyVoice 3

## 定义

阿里通义 [[CosyVoice 2]] 的后继版本，**0.5B / 1.5B 双尺寸**，训练数据 **1M 小时**（9 语言 + 18 中国方言）。核心创新：(1) 基于 [[MinMo]] 的监督多任务 [[Speech Tokenizer]]（[[FSQ]] 量化，25 Hz）；(2) [[DiffRO]] 后训练（Gumbel-Softmax 使离散 token 采样可微）；(3) CFM 升级为 [[DiT]] backbone（300M）。

## 核心要点

1. **架构**: LLM (0.5B/1.5B) AR 预测 speech token → DiT-based [[Flow Matching|CFM]] (300M) → Vocoder
2. **Tokenizer**: 基于 MinMo 的 53 万小时五任务（ASR/LID/SER/AED/SA）联合训练，FSQ 量化
3. **后训练**: DiffRO 在 speech token 空间做可微 RL，绕过 CFM/Vocoder，通用适用于 LLM-based TTS
4. **数据**: 100 万小时，6 步数据管线（检测→降噪→三模型交叉 ASR→标点→音量→过滤）
5. **结果**: SEED-TTS-eval test-zh CER 0.71%（CosyVoice 2: 1.45%，↓51%），test-en WER 1.45%（↓44%）；MOS 英文达人类水平
6. **覆盖**: 唯一覆盖 9 语言的零样本 TTS 系统

## 代表工作

- [[CosyVoice3]]: CosyVoice 3 论文笔记

## 评测/常见数字

- SEED-TTS-eval test-zh CER: 0.71%（1.5B+RL），test-en WER: 1.45%
- CV3-Eval 9 语言全覆盖，DiffRO 带来 20-50% 相对 WER 提升
- MOS: 中文 4.50+，英文达/超人类

## 相关概念

- [[CosyVoice]] / [[CosyVoice 2]]: 前置版本
- [[Qwen3-TTS]]: 同阿里出品但走纯 LLM TTS 路线
- [[Seed-TTS]]: 字节同档对手
- [[FSQ]]: 核心量化方法
- [[DiffRO]]: 核心后训练方法
- [[MinMo]]: Tokenizer 基座模型
