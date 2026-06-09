---
type: concept
aliases: [Self-corrective Alignment, 自纠正对齐, Reward-free Self-correction]
---

# SOAR

## 定义
Self-corrective alignment（自纠正对齐），一种 reward-free 的后训练方法。核心思想：让 flow-matching / diffusion 模型在训练时暴露于自身推理轨迹的偏移状态（off-trajectory），学习从这些偏移状态正确回归 clean target，从而弥合训练-推理的分布差异。

## 核心要点
1. 从当前时间步做 detached Euler rollout 一步，产生 off-trajectory 状态
2. 对 off-trajectory 状态 re-noise 生成多个辅助训练点
3. 不需要 reward model、人类偏好数据或外部 teacher
4. 只更新生成网络（如 DiT），其他模块冻结

## 代表工作
- [[dots-tts]]: 首次将 SOAR 应用于 AR flow-matching head，50K steps 后训练
- SOAR 原始论文: 应用于图像/视频 diffusion 模型

## 相关概念
- [[Flow Matching]]
- [[Classifier-Free Guidance]]
- [[Speech DPO]]
- [[DiffRO]]
