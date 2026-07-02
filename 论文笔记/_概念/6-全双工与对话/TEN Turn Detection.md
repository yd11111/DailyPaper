---
type: concept
aliases: [TEN Turn, Ten Turn Detection]
---

# TEN Turn Detection

## 定义
TEN Framework 中的 turn detection 模块，依赖 Paraformer ASR 转录文本做 turn 预测。需要完整 ASR 前置，在噪声和重叠语音下性能下降。

## 核心要点
1. 依赖 Paraformer ASR 提供转录文本
2. Paraformer (~7220M 参数) + TEN Turn 的组合参数量大
3. 噪声和重叠语音下 ASR 质量下降导致 turn 判断性能退化
4. 开源实现：https://github.com/TEN-framework/ten-turn-detection

## 代表工作
- [[FastTurn]]: 对比基线

## 相关概念
- [[Turn-Taking]]
- [[Paraformer]]
