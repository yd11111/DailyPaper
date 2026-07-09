---
type: concept
aliases: [CDM, 通道复用, Channel Division Multiplexing]
domain: Dialogue
tags: [full-duplex, architecture-paradigm]
created: 2026-07-09
---

# Channel-Division Multiplexing (CDM)

## 定义

全双工语音交互的一种建模范式：为输入音频流（listening）和输出音频流（speaking）分配**独立的通道**，在统一模型内并行处理。区别于 Time-Division Multiplexing（TDM，将听/说 token 展平成单一时序序列）。

## 核心要点

1. CDM 是最"集成"的全双工方案——单一模型同时编码输入和解码输出，无需外部 VAD 或多阶段流水
2. 核心挑战：共享参数空间中模态纠缠导致知识退化（梯度冲突）
3. 代表做法：[[Moshi]] 的双流 AR、[[Lychee-FD]] 的分层分离

## 代表工作

- [[Moshi]]: 首个大规模 CDM 全双工系统，全共享参数
- [[Lychee-FD]]: 浅共享+深分离的改进 CDM
- [[SALMONN-omni]]: CDM 路线的 Native 端到端方案

## 相关概念

- [[Time-Division Multiplexing]]
- [[梯度冲突]]
- [[全双工]]
