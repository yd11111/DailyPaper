---
type: concept
aliases: [Qwen3-Omni, Qwen3-Omni-30B-A3B]
---

# Qwen3-Omni

## 定义

阿里 Qwen 系列的 omni-modal 大模型，30B-A3B (30B 总参数 / 3B 激活的 MoE)，覆盖 vision + audio + text 三模态。是 [[MiniCPM-o 4.5]] 主要直接竞品。

## 核心要点

- **MoE 架构**: 30B 总参数，3B 激活
- 强 audio understanding（继承自 [[Qwen2-Audio]] / Qwen2.5-Omni 谱系）
- 部分 benchmark（AISHELL-2、WenetSpeech test-net、MMAU、Speech Web Questions）领先
- 端侧部署门槛较高（INT4 RTX 4090 显存 20 GB、首 token 0.98 s、BF16 OOM）

## 相关版本

- Qwen2.5-Omni (2025/03)
- Qwen3-Omni (2025/09)

## 相关概念

- [[MiniCPM-o]]: 同期 9B 开源对手
- [[Qwen3-VL]]: 同系列纯视觉版
- [[Qwen3-8B]]: 同系列纯文本 backbone

## 评测/常见数字

| 任务 | 数字 | vs MiniCPM-o 4.5 |
|---|---|---|
| AISHELL-2 ↓ | 2.3 | 优于 (2.5) |
| WenetSpeech test-net ↓ | 4.7 | 优于 (5.9) |
| OpenCompass | 75.7 | 略低 (77.6) |
| OmniDocBench EN ↓ | 0.216 | 远高于 (0.109) |
| RTX 4090 INT4 throughput | 147.8 tok/s | 远低于 (212.3) |
| RTX 4090 INT4 memory | 20 GB | 远高于 (11 GB) |
