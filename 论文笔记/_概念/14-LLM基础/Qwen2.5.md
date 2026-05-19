---
type: concept
aliases: [Qwen2.5-1.5B, Qwen2.5-7B]
---

# Qwen2.5

## 定义

阿里 2024 年发布的 Qwen2.5 通用 LLM 系列。从 0.5B 到 72B 多档；强多语种 + 强代码能力，开源。常被当作多模态模型的 backbone（音频/视频/视觉端）。

## 核心要点

1. 32K 原生 context、可外推
2. RoPE + GQA + RMSNorm
3. **VibeVoice 用 1.5B / 7B 两版**作为 next-token diffusion 的主干，并通过 curriculum 把 context 扩到 64K
4. Qwen2.5 系列也是 Qwen2.5-Omni / Qwen2-Audio 的语言端基座

## 代表工作

- 原 tech report (arXiv 2412.15115)
- [[VibeVoice]]: 1.5B & 7B 主干

## 相关概念

- LLM
- [[VibeVoice]]
