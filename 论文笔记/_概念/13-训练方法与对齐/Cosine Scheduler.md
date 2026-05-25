---
type: concept
aliases: [Cosine Schedule, 余弦调度]
---

# Cosine Scheduler

## 定义
将均匀采样的时间步 $t$ 通过余弦函数重映射，使扩散/Flow Matching 过程在生成初期（噪声较大时）分配更多步数。

## 数学形式

$$
t := 1 - \cos\left(\frac{1}{2} t \pi\right)
$$

## 核心要点
1. 在 Flow Matching 中，生成初期的去噪最困难，需要更细粒度的步长
2. CosyVoice 中用于 OT-CFM 的时间步重映射
3. 也常见于 DDPM 的噪声调度（cosine noise schedule）

## 代表工作
- [[CosyVoice]]: Flow Matching 的 cosine timestep 调度

## 相关概念
- [[Conditional Flow Matching]]
- [[DDPM]]
