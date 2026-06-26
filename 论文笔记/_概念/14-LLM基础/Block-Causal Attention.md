---
type: concept
aliases: [Block Causal Attention, 块因果注意力]
---

# Block-Causal Attention

## 定义
一种介于 token-level causal attention 和 full attention 之间的注意力机制：将序列划分为固定大小的 block（如对应一个 streaming unit），block 内的 token 可以互相 attend（双向），但 block 间严格保持因果性（只能看到过去的 block）。

## 核心要点
1. 相比 token-level causal attention，block 内 token 可以利用同 block 的未来信息，提高表达能力
2. 相比 full attention，保持了 block 间的因果性，支持流式推理
3. 特别适合多模态流式生成：一个 block 内的文本/音频/视频 token 可以互相 attend，实现 unit 内跨模态信息交换

## 代表工作
- [[Wan-Streamer]]: 用 block-causal attention 实现 160 ms streaming unit 的多模态流式生成
- [[Moshi]]: 类似理念，用于全双工语音对话

## 相关概念
- [[KV Cache]]
- [[Full-Duplex]]
- [[Sliding-Window Attention]]
