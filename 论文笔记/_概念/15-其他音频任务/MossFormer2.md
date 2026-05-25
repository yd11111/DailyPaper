---
type: concept
aliases: [MossFormer 2, 语音增强模型]
---

# MossFormer2

## 定义

结合 Transformer 和 RNN-free 循环网络的语音增强/分离模型，用于从含噪音频中提取干净语音。

## 核心要点

1. 混合架构：Transformer 全局注意力 + 循环网络局部建模
2. 无需 RNN 的循环机制，推理效率高
3. 在 CosyVoice 3 数据管线中用于百万小时互联网音频的降噪预处理

## 代表工作

- Zhao et al. 2024: "MossFormer2: Combining transformer and RNN-free recurrent network"
- [[CosyVoice3]]: 数据管线降噪步骤

## 相关概念

- [[Source Separation]]
- [[VAD]]
