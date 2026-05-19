---
type: concept
aliases: [Half-duplex Dialogue Training, 半双工对话训练]
---

# Half-duplex Dialogue Training

## 定义
用轮替式（turn-based）对话数据训练语音对话模型的阶段，无 overlapping speech，相对简单。

## 核心要点
1. User 说完 Assistant 才说，无并发
2. 数据分布与 ASR/TTS 一致，模型适应平滑
3. [[Curriculum Learning]] 中 full-duplex 的前置阶段

## 代表工作
- [[OmniFlatten]] 阶段 2，证明加入半双工 SFT 显著提升 full-duplex 性能

## 相关概念
- [[OmniFlatten]]
