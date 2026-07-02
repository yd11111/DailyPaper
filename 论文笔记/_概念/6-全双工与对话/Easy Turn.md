---
type: concept
aliases: [EasyTurn]
---

# Easy Turn

## 定义
一种基于 ASR 转录文本的 turn detection 方法，通过先做完整语音识别再分析文本语义来判断 turn 状态。准确度较高但延迟较大。

## 核心要点
1. ASR-first 流水线：先完成语音识别，再基于文本做 turn 预测
2. 支持多类别 turn 状态检测（Complete / Incomplete / Backchannel / Wait）
3. 延迟约 300-350ms，受限于 ASR 前置处理
4. 对复杂声学信息（噪声、重叠语音）建模能力有限

## 代表工作
- [[FastTurn]]: 对比基线，用流式 CTC 替代完整 ASR 降低延迟

## 相关概念
- [[Turn-Taking]]
- [[VAD]]
- [[Full-Duplex]]
