---
type: concept
aliases: [VITA 1.5, VITA]
domain: Omni
tags: [omni-model, multimodal, system-level]
created: 2026-07-09
---

# VITA-1.5

## 定义

腾讯出品的多模态交互模型。在全双工场景下属于 System-level 方案——外接 VAD 驱动半双工 SLM，保留强语义能力但延迟较高。

## 核心要点

1. System-level 全双工方案（VAD + 半双工 SLM）
2. Spoken QA Avg S→T = 50.8%（强语义基线）
3. Avg S→S = 35.4%（文本到语音的降质较明显）
4. TOR = 100（总是能回答），但 FSED/IRD 延迟高

## 代表工作

- [[Lychee-FD]]: 以其作为 System-level 上限基线

## 相关概念

- [[全双工]]
- [[Freeze-Omni]]
