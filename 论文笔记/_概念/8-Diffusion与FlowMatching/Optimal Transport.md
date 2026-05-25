---
type: concept
aliases: [OT, 最优传输]
---

# Optimal Transport

## 定义
最优传输理论，寻找从一个概率分布到另一个分布的最低代价映射。在 Flow Matching 中用于构造直线传输路径，加速收敛。

## 数学形式

$$
\phi_t^{OT}(X_0, X_1) = (1-(1-\sigma)t)X_0 + tX_1
$$

## 核心要点
1. OT 路径比随机耦合路径更直、梯度更简单
2. OT-CFM 是 CosyVoice 和多个 TTS 系统的核心生成范式
3. 理论上保证更快的训练收敛

## 代表工作
- [[CosyVoice]]: OT-CFM 生成 Mel
- [[Voicebox]]: Flow Matching with OT

## 相关概念
- [[Conditional Flow Matching]]
- [[OT-CFM]]
- [[Flow Matching]]
