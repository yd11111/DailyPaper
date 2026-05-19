---
type: concept
aliases: [LSLM, Listen while Speaking]
---

# LSLM

## 定义
上交 Ma et al. 2024 提出，让模型 "边听边说" 的方案；通过外接 TTS 实现 turn-taking。

## 核心要点
1. Listening LM 在 generation 时持续监听 user 语音
2. 随时根据 listener 决策停止 / 继续生成

## 代表工作
- [[OmniFlatten]] 视其为 full-duplex 早期探索

## 相关概念
- [[OmniFlatten]]
