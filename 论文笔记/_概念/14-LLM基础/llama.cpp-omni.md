---
type: concept
aliases: [llama.cpp-omni]
---

# llama.cpp-omni

## 定义

[[MiniCPM-o 4.5]] 团队基于 [[llama.cpp]] 二次开发的端侧推理框架，**专为 streaming 全双工 omni 交互优化**。在 macOS / Windows / Linux 上提供低 RTF / 低显存的实时部署。

## 核心要点

1. **流式调度优化**: 配合 [[Omni-Flow]] 的 chunk-wise 推理节奏
2. **多硬件**: RTX 4090 / DGX Spark / 消费级 CPU+GPU
3. **INT4 量化**: 显著降低显存与延迟

## 评测/常见数字

| 框架 | Dtype | RTX 4090 RTF↓ | 显存(GB)↓ | DGX Spark RTF↓ |
|---|---|---|---|---|
| PyTorch | BF16 | OOM | OOM | 2.43 |
| PyTorch | INT4 | 1.26 | 14 | 1.27 |
| **llama.cpp-omni** | FP16 | 0.27 | 19 | 0.46 |
| **llama.cpp-omni** | INT4 | **0.21** | **11** | **0.20** |

## 相关概念

- [[llama.cpp]]
- [[GGUF]]
- [[MiniCPM-o]]
- [[Edge Deployment]]
