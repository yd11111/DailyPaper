---
type: concept
aliases: [OT-CFM, Optimal-transport Conditional Flow Matching]
---

# OT-CFM

## 定义
基于最优传输的 Conditional Flow Matching 生成模型，由 Lipman et al. 2023 与 Tong et al. 2024 提出。

## 核心要点
1. 比 diffusion 训练更简单（梯度更直接）
2. 采样步数少，推理更快
3. TTS 中常用作 token → mel 的生成器
4. 本质：学一个 vector field 从 noise 到 data 的最优传输路径

## 代表工作
- [[CosyVoice]] / [[OmniFlatten]] / [[F5-TTS]] / [[Voicebox]] 都用 Flow Matching 范式

## 相关概念
- [[OmniFlatten]]
