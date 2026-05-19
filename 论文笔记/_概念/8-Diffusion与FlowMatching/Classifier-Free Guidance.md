---
type: concept
aliases: []
---

# Classifier-Free Guidance

## 定义

扩散/Flow 模型推理引导技巧：训练时随机 drop 条件，得到无条件分支；推理时按 $\hat{\epsilon} = (1+w)\epsilon_c - w\epsilon_\emptyset$ 组合。SemaVoice 取 $w=2.5$。

## 核心要点

1. （待补充）

## 代表工作

- [[SemaVoice]]
- [[VibeVoice]]: 推理时 CFG scale $w=1.3$，配合 [[DPM-Solver|DPM-Solver++]] 10 步采样

## 相关概念

[[DDPM]] / [[Flow Matching]] / [[DPM-Solver]] / [[VibeVoice]]
