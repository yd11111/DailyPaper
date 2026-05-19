---
type: concept
aliases: [SyncLLM, SyncLM]
---

# SyncLLM

## 定义
华盛顿大学 Veluri et al. 2024 年提出的端到端全双工语音对话模型，采用 chunk-based 时间同步策略。

## 核心要点
1. 时间分块 (time-chunking) + 去重策略 (deduplication) 简化建模
2. 去重导致音频重建有误差
3. 只产 speech，不产 text

## 代表工作
- [[OmniFlatten]] 与之最相近的并发工作；OmniFlatten 强调保留完整 speech token + 同时产 text 是更优选择

## 相关概念
- [[OmniFlatten]]
