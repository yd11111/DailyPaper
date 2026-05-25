---
type: concept
aliases: [条件生成, Conditional Generative Model]
---

# Conditional Generation

## 定义
在生成模型中引入额外条件信息来控制生成内容属性的技术。在 TTS 中，条件可以是文本/语言学特征（局部条件）或说话人身份（全局条件）。

## 核心要点
1. **全局条件**: 单一向量（如说话人 embedding）广播到所有时间步，控制整体属性
2. **局部条件**: 时变序列（如语言学特征）经上采样后逐帧注入，控制局部内容
3. WaveNet 通过将条件信号加入 [[Gated Activation Unit]] 的两个分支实现
4. 后续 TTS 模型普遍采用类似条件注入机制（如 [[Tacotron 2]] 的 attention、[[VALL-E]] 的 prompt）

## 代表工作
- [[WaveNet]]: 全局条件（说话人 one-hot）+ 局部条件（语言学特征 + F0）

## 相关概念
- [[Gated Activation Unit]]
- [[Autoregressive Model]]
