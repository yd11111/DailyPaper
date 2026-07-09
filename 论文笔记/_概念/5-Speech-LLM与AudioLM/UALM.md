---
type: concept
aliases: [Unified Audio Language Model]
---

# UALM

## 定义

NVIDIA 提出的统一音频语言模型（1.5B/7B），首次在 LLM 框架中用 BPE audio token + Classifier-Free Guidance 实现 text-to-audio 生成质量接近专用 diffusion 模型，同时保持音频理解和文本能力。是 Audex 的前身。

## 核心要点
1. 用 LLM 的 next-token prediction 范式统一音频理解与生成
2. CFG 是 TTA 质量的关键：不用 CFG 质量远不如 diffusion 基线
3. 用 BPE token（非 cross-attention embedding）条件化文本输入
4. 证明了 proper data blending + CFG 能弥合 LLM-based 与 diffusion-based TTA 的质量差距

## 代表工作
- [[Audex]]: UALM 的后续，扩展到 30B MoE + 加入 TTS/ASR/S2S + Cascade RL
- Tian et al., 2026: UALM 原始论文

## 相关概念
- [[Classifier-Free Guidance]]
- [[X-Codec]]
- [[Audex]]
