---
type: concept
aliases: []
---

# BigCodec

## 定义

大规模单 codebook 神经语音 codec。通过扩大模型规模（encoder/decoder 参数量）在仅 1 个 codebook 条件下实现与 [[RVQ]] 多 codebook codec 可比的重建质量，简化了下游 Speech LLM 的 token 序列建模。

## 核心要点

1. 单 codebook 设计，降低 Speech LLM 建模复杂度（无需多层 RVQ 展开）
2. 通过增大 encoder/decoder 容量弥补信息压缩损失
3. 常被用作低码率 codec 的对比 baseline（如 [[CFMDCTCodec]] 在 0.65 kbps 场景对比）

## 代表工作

- BigCodec (2024)
