---
type: concept
aliases: [Freeze Omni]
domain: Dialogue
tags: [full-duplex, system-level, voice-agent]
created: 2026-07-09
---

# Freeze-Omni

## 定义

System-level 全双工语音交互方案：冻结 LLM backbone，外接 VAD 作为对话管理器驱动半双工 SLM 进行轮流对话。以保留知识能力为代价换取模型稳定性。

## 核心要点

1. 属于 System-level 方案（VAD + 半双工 SLM）
2. 优势：知识保留好（LLM backbone 不动）
3. 劣势：VAD 错误级联、延迟高（FSED 667ms）、无法真正同时听说

## 代表工作

- [[Lychee-FD]]: 以其作为 System-level 基线对比

## 相关概念

- [[VAD]]
- [[全双工]]
- [[Channel-Division Multiplexing]]
