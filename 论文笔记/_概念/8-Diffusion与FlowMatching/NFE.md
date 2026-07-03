---
type: concept
aliases: [Number of Function Evaluations, 函数评估次数, ODE Steps]
---

# NFE

## 定义
NFE（Number of Function Evaluations）是衡量 Diffusion / Flow Matching 模型推理成本的核心指标，表示求解 ODE/SDE 时网络前向传播的次数。NFE 越低推理越快，但过低会导致生成质量下降。

## 核心要点
1. 标准 Diffusion 模型 NFE 通常 20-1000 步，Flow Matching 模型通常 5-20 步
2. 降低 NFE 的方法：[[Rectified Flow]]（矫直 ODE 路径）、[[Consistency Model]]（一步生成）、知识蒸馏
3. 使用 [[Classifier-Free Guidance]] 时，每步 NFE 实际需要 2 次前向传播（条件 + 无条件）
4. NFE 直接影响 [[RTF]]（Real-Time Factor）

## 代表工作
- [[UnifiedGuidanceFM]]: 将 NFE 从 10 步降至 3 步（且无 CFG 双传播），RTF 从 0.078 降至 0.024
- [[F5-TTS]]: 使用 ~32 步 ODE 采样
- [[Rectified Flow]]: 通过矫直 ODE 路径减少 NFE

## 相关概念
- [[Flow Matching]]
- [[Rectified Flow]]
- [[RTF]]
- [[Classifier-Free Guidance]]
