---
type: concept
aliases: [Full-duplex Dialogue Training, 全双工对话训练]
---

# Full-duplex Dialogue Training

## 定义
用支持同时听说的对话数据训练模型，需处理 turn-taking、barge-in、overlapping speech 等真实交互动态。

## 核心要点
1. 需要多通道（user / assistant）合成或真实数据
2. 通常引入 chunk-based 时间对齐
3. OmniFlatten 进一步细分为 3-stream 和 2-stream 子阶段

## 代表工作
- [[OmniFlatten]] 阶段 3，3-stream 保留 assistant text 引导语义，2-stream 进一步去文本

## 相关概念
- [[OmniFlatten]]
