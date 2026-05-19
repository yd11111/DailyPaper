---
type: concept
aliases: [Moshi, Kyutai Moshi]
---

# Moshi

## 定义
Kyutai 团队 2024 年提出的端到端**全双工**语音对话模型，被认为是低延迟语音 LLM 的标杆。

## 核心要点
1. 多流并行建模：user speech / assistant text / assistant speech 同时预测
2. 采用 **acoustic delay** + **inner monologue**（让模型先想文本再说话）让 AR 模型能并行多流
3. 内置 [[Mimi]] codec（12.5 Hz, 1.1 kbps），专为 LLM 设计
4. 端到端延迟约 160 ms，业内首次实现真正自然的 full-duplex 对话

## 代表工作
- [[OmniFlatten]] 与之对比，强调用 flatten 单流避免其多流定制设计

## 相关概念
- [[OmniFlatten]]
