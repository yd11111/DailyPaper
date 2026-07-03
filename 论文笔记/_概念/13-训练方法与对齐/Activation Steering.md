---
type: concept
aliases: [激活引导, Representation Engineering, 表示工程, Activation Engineering]
domain: LLM基础
tags: [controllability, inference-time-intervention, activation-editing]
created: 2026-07-03
---

# Activation Steering

## 定义

在推理时向模型中间层的激活向量注入学习到的方向向量（steering vector），以控制模型输出的特定属性（如情感、风格、安全性等），无需重新训练模型。

## 数学形式

$$
\tilde{\mathbf{h}}^{(l)} = f_r\!\left(\mathbf{h}^{(l)} + \alpha \cdot \mathbf{v}^{(l)}\right)
$$

其中 $\mathbf{v}^{(l)}$ 是目标属性的方向向量（通常为目标属性样本与中性样本的激活差均值），$\alpha$ 控制强度，$f_r$ 做 renormalization。

## 核心要点

1. 相比 fine-tuning，activation steering 不改变模型参数，可以在推理时动态组合多个属性方向
2. 核心假设：目标属性在模型中间层有线性可分的子空间（linear representation hypothesis）
3. 可组合性取决于不同属性方向的独立性——可用 ΔLID 等几何指标评估
4. 在 LLM 安全控制（Zou et al., 2023）和文本到图像生成（Rodriguez et al., 2025）中已有成功应用

## 代表工作

- Zou et al. (2023): Representation engineering，LLM 透明性
- Turner et al. (2023): Activation engineering for LLM steering
- Rimsky et al. (2024): Contrastive activation addition for Llama 2
- [[CoCoEmo]]: 在 TTS SLM 上做混合情感 steering
- [[EmoSteer-TTS]]: 在 TTS CFM 上做单情感 steering
- [[GeometricEmotionSteering]]: 几何分析 SLM vs CFM 的 steering 适合度

## 相关概念

- [[Linear Probing]]
- [[Local Intrinsic Dimensionality]]
- [[Classifier-Free Guidance]]
