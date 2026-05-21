---
type: concept
aliases: [Interpausal Unit, IPU, 间停单元]
---

# IPU (Interpausal Unit)

## 定义

**Interpausal Unit**：对话语音分析中的基本时间单位，定义为同一说话人**两段足够长静音之间的连续语音段**。是对话动力学（turn-taking、entrainment、prosody）分析的最小事件单位。

## 数学形式

给定单个说话人的帧级语音活动序列 $v_t \in \{0, 1\}$ 和静音阈值 $\tau$（常用 80–200 ms）：

- 一个 IPU 是最长的连续帧区间 $[t_s, t_e]$，使得 $v_{t}=1$ 对 $t \in [t_s, t_e]$ 成立；
- 相邻 IPU 之间静音长度 $\geq \tau$。

[[Synchronization-Turn-Taking]] 用的阈值是 **80 ms**。

## 核心要点
1. **比"句子"更细**：IPU 是声学事件，不依赖文本切分，因此可以用纯 [[VAD]] 提取
2. **比"帧"更稳**：避免帧级噪声，给 [[Turn-taking]] / [[EOI Prediction]] 提供合适的时间锚
3. **静音阈值是超参**：不同论文用 80/180/200 ms 不等，结果不可直接比较
4. **承担 turn 决策的离散事件**：[[Hold vs Non-Hold]] 等任务标签都打在 IPU 边界上

## 代表工作
- [[Synchronization-Turn-Taking]]: 用 80 ms 静音阈值切 IPU，作为 [[EOI Prediction]] 与 [[Hold vs Non-Hold]] 探针的时间单位

## 相关概念
- [[VAD]]
- [[Turn-taking]]
- [[EOI Prediction]]
- [[Hold vs Non-Hold]]
