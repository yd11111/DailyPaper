---
type: concept
aliases: [SmartTurn]
---

# Smart Turn

## 定义
Pipecat-AI 开源的轻量 turn detection 方法，使用简单线性层做 turn 预测。参数量极小（~32M），速度快但处理复杂场景能力有限。

## 核心要点
1. 极轻量设计，仅 ~32M 参数
2. 仅支持 2 类分类（Complete / Incomplete）
3. 推理延迟低（~70ms）
4. 对 backchannel、噪声等复杂场景处理能力有限

## 代表工作
- [[FastTurn]]: 对比基线

## 相关概念
- [[Turn-Taking]]
- [[Easy Turn]]
