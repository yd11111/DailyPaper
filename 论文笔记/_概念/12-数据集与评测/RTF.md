---
type: concept
aliases: [Real-Time Factor, 实时因子]
---

# RTF

## 定义

**Real-Time Factor（实时因子）**：处理 1 秒音频所需的实际墙钟时间。$\text{RTF} < 1$ 才能实时；$\text{RTF} = 0.1$ 表示 1 秒音频用 0.1 秒处理（10× 实时）。

## 数学形式

$$
\text{RTF} = \frac{T_{compute}}{T_{audio}}
$$

## 核心要点

1. 是流式 [[ASR]] / [[TTS]] / [[Full-Duplex|全双工]] 系统的核心硬性指标。
2. 单 batch 通常报告：batched 推理下的 RTF 会显著降低，但不代表真实在线延迟。
3. 与"首包延迟"（first-packet latency）配套报告才有意义——RTF 0.05 但首包 5 秒也没法做对话。
4. GPU / CPU、batch size、序列长度都会影响，论文需注明硬件配置。

## 评测/常见数字

- 实时云端 ASR：典型 RTF 0.03–0.2（GPU）
- 流式 TTS：RTF < 0.3 + 首包 < 300ms 视为可对话
- [[Whisper]]-Large（batch=1）：RTF ~0.1–0.3（A100）

## 相关概念

- [[ASR]]
- [[TTS]]
- [[Full-Duplex]]
- [[WER]]
