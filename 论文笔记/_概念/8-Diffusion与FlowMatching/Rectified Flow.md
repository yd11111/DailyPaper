---
type: concept
aliases: [直线流, Linear Flow, Straight-line Flow]
---

# Rectified Flow

## 定义
一种 [[Flow Matching]] 的具体实例化方式。采用直线插值路径 $x_t = (1-t)x_0 + tx_1$ 连接噪声 $x_0 \sim \mathcal{N}(0,I)$ 和数据 $x_1$，对应的目标速度场为常数 $v^* = x_1 - x_0$。

## 数学形式

$$
x_t = (1-t)\epsilon + t x_1, \quad v^*(x_t, t) = x_1 - \epsilon
$$

训练目标：$\mathcal{L} = \mathbb{E}\left[\|v_\theta(x_t, t) - (x_1 - \epsilon)\|^2\right]$

## 核心要点
1. 路径直线 → ODE 轨迹更平滑，少步推理质量更高
2. 与 DDPM 的 $\epsilon$-prediction 等价（通过简单变换）
3. 现代 TTS/Audio 系统的主流选择

## 代表工作
- [[dots-tts]]: AR-FM Head 使用 rectified flow 目标
- [[F5-TTS]]: 纯 flow matching TTS
- [[Voicebox]]: 条件 flow matching

## 相关概念
- [[Flow Matching]]
- [[Conditional Flow Matching]]
- [[DDPM]]
