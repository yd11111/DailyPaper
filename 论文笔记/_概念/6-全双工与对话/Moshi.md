---
type: concept
aliases: [Moshi-v2]
---

# Moshi

## 定义

Kyutai Labs 2024 年的全双工语音对话模型。**双流 token 同时建模用户与模型音频**，支持随时打断（barge-in）、自然 turn-taking；端到端延迟 ~160 ms。配套使用低码率 codec [[Mimi]] (12.5 Hz / 1.1 kbps)。

## 核心要点

1. **双流 (dual-stream) AR**：模型与用户两条 token 流并行预测
2. 全双工：边听边说，无需 VAD
3. 极低延迟：~160 ms 首包
4. 是 [[VibeVoice]] 的 future work 暗示方向（处理 overlap / 抢话）

## 代表工作

- 原论文 (Kyutai 2024)
- [[VibeVoice]]: 在 limitations 中引用 Moshi 作为对照（VibeVoice 不显式建模 overlap）

## 评测/常见数字

- 端到端延迟: ~160 ms
- Mimi codec: 12.5 Hz / 1.1 kbps

## 相关概念

- [[Audio Codec]]
- 全双工对话
