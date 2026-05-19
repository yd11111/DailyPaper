---
type: concept
aliases: [LLaMA-Omni]
---

# LLaMA-Omni

## 定义
中科院计算所 2024 年提出的端到端语音对话模型，基于 LLaMA-3.1-8B + speech encoder + streaming TTS。

## 核心要点
1. 8B 参数，半双工 turn-based
2. Speech encoder 直接接 LLaMA，绕过离散 speech token
3. Streaming TTS 模块直接生成 speech，避免 vocoder 延迟
4. 中文能力弱（dialogue learning 没用中文数据）

## 代表工作
- [[OmniFlatten]] 在英文 chat 上落后 LLaMA-Omni，但中文反超

## 相关概念
- [[OmniFlatten]]
