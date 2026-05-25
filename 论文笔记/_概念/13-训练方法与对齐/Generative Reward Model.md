---
type: concept
aliases: [GRM, Generative Reward Model, 生成式奖励模型]
---

# Generative Reward Model

## 定义

一种奖励模型变体，不同于传统标量 reward model 只输出单一分值，GRM 以生成式方式评估候选响应相对于参考响应的质量，能捕获更细粒度的人类偏好信号。

## 核心要点

1. 给定 prompt $x$、候选响应 $y$ 和高质量参考 $y^*$，GRM $r_\phi$ 产生成对偏好得分 $r_\phi(x, y, y^*)$。
2. 最终奖励经 shaping 变换：$r_{hf}(x, y, y^*) = s(r_\phi(x, y, y^*))$。
3. 比标量 reward model 更适合评价语音合成质量、对话自然度等多维属性。
4. [[Step-Audio-2.5]] 在 TTS 和 Realtime 两个分支中共用 GRM-based [[RLHF]]。

## 代表工作

- StepAudio 2.5 (StepFun, 2026): TTS + Realtime 共用 GRM-PPO

## 相关概念

- [[RLHF]]
- [[Speech DPO]]
- [[Step-Audio-2.5]]
