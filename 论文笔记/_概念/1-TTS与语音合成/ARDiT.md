---
type: concept
aliases: [Autoregressive Diffusion Transformer]
---

# ARDiT

## 定义
Autoregressive Diffusion Transformer，一种 decoder-only 架构将自回归生成与 diffusion/flow-matching 结合。每个 AR 步用 DiT 做 per-patch 的 diffusion 生成，而非预测离散 token。

## 核心要点
1. Decoder-only DiT 同时处理 AR 序列建模和 per-step 扩散生成
2. 启发了 dots.tts 的三模块解耦设计——将 ARDiT 的单体模型拆为 LLM + AR-FM Head
3. AR-FM Head 携带完整 LLM hidden history，本质上是完整的 text-conditioned 生成器

## 代表工作
- ARDiT 原始论文
- [[dots-tts]]: 受 ARDiT 启发的三模块解耦设计

## 相关概念
- [[DiT]]
- [[Continuous Autoregressive TTS]]
- [[Flow Matching]]
