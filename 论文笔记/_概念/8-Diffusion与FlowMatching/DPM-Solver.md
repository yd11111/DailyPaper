---
type: concept
aliases: [DPM-Solver++, DPMSolver, DPM-Solver-2/3]
---

# DPM-Solver

## 定义

清华朱军组 2022/2025 年提出的扩散模型 ODE 数值解算器系列。把扩散反向 SDE 转成 ODE 后，用半线性结构 + 二阶/三阶 Taylor 展开高效采样；典型 10–20 步即达 1000 步质量。**DPM-Solver++** 为带 [[Classifier-Free Guidance|CFG]] 引导采样进一步优化。

## 核心要点

1. 在 1000-step DDPM 上把推理压到 ~10 步
2. **DPM-Solver++** 专为 guided sampling 设计；[[VibeVoice]] 用它做 token-level 扩散头的 10 步采样
3. 与 EDM、Euler、Heun 等同属现代扩散加速器

## 代表工作

- DPM-Solver 原论文 (Lu et al. NeurIPS 2022)
- DPM-Solver++ 原论文 (Lu et al. MIR 2025)
- [[VibeVoice]]: 推理阶段使用 DPM-Solver++ 10 步

## 相关概念

- [[DDPM]]
- [[Classifier-Free Guidance]]
- [[Flow Matching]]
