---
type: concept
aliases: [空洞因果卷积, Dilated Causal Conv, à trous causal convolution]
---

# Dilated Causal Convolution

## 定义
在 [[Causal Convolution]] 基础上引入空洞率（dilation rate），以指数增长的间隔采样输入，从而在保持因果性和计算效率的同时，实现指数级感受野增长。

## 数学形式

$$
y_t = \sum_{k=0}^{K-1} w_k \cdot x_{t - d \cdot k}
$$

- $d$: 空洞率（dilation rate），按层指数增长，如 $d = 1, 2, 4, 8, \dots, 512$
- 感受野: 单个 block ($d = 1, 2, \dots, 2^{N-1}$) 的感受野为 $2^N$
- 多 block 堆叠进一步扩大感受野和模型容量

## 核心要点
1. WaveNet 的核心架构创新，解决了普通因果卷积感受野增长慢的问题
2. 一个 $d=1,2,\dots,512$ 的 block 感受野为 1024，等效于 $1 \times 1024$ 卷积的非线性高效替代
3. 多 block 重复堆叠（如 3 个 block）进一步增加容量
4. 不增加参数量和计算量即可获得大感受野

## 代表工作
- [[WaveNet]]: 提出并首次应用于原始音频波形生成

## 相关概念
- [[Causal Convolution]]
- [[Dilated Convolution]]
- [[TCN]]
