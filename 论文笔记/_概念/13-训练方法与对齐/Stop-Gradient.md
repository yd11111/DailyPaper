---
type: concept
aliases: [sg, stop-grad, detach, 梯度截断]
---

# Stop-Gradient

## 定义
Stop-Gradient（也写作 sg(·) 或 detach()）是一种训练技巧，将某个张量从计算图中分离，使梯度不通过该张量反向传播。在蒸馏、对比学习、EMA 更新等场景中广泛使用，核心作用是控制梯度流向，防止某些分支的参数被不期望的梯度信号更新。

## 数学形式

$$
\operatorname{sg}(x) = x \quad (\text{前向不变})
$$

$$
\frac{\partial \operatorname{sg}(x)}{\partial x} = 0 \quad (\text{反向梯度为零})
$$

## 核心要点
1. 前向计算时 sg(x) 等于 x 本身，不改变数值
2. 反向传播时梯度被截断，不流过 sg(x) 对应的分支
3. 在 PyTorch 中通常用 `.detach()` 或 `torch.no_grad()` 实现
4. 在 TensorFlow/JAX 中通常用 `tf.stop_gradient()` 或 `jax.lax.stop_gradient()` 实现

## 代表工作
- [[UnifiedGuidanceFM]]: 在 CFG 蒸馏的引导速度场目标中使用 sg() 冻结 CFG 方向
- BYOL / SimSiam: 对比学习中用 stop-gradient 防止模式坍塌
- EMA Teacher: 知识蒸馏中 teacher 网络不接受梯度

## 相关概念
- [[Knowledge Distillation]]
- [[Classifier-Free Guidance]]
