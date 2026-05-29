---
type: concept
aliases: [Gemini 3 Pro, Gemini-3-Pro, Gemini-3]
domain: Omni
tags: [omni, lalm, audio-judge, closed-source, google]
related_maps:
  - "[[Omni-代表模型谱系]]"
  - "[[TTS-评测体系]]"
created: 2026-05-29
last_updated: 2026-05-29
---

# Gemini 3 Pro

## 定义

Google DeepMind 2025-12 发布的 Gemini 3 系列旗舰多模态模型，支持音频输入。论文标题"Gemini 3"，官方页 <https://deepmind.google/technologies/gemini/>。

## 核心要点

1. **音频理解能力为 2025-12 时点最强的闭源 LALM**：[[SwanBench-Speech]] §D.4 在 12 个 evaluator（4 MOS 网络 + 8 LALM 包括 GPT-4o / Qwen3-Omni-Instruct/Flash / StepFun-Audio-R1 / Gemini-2.5-flash/pro / Gemini-3-flash）的对齐分横评中**Richness/Hierarchy 双榜第一**
2. **稳定性强**：5 次重复评测仅 11/200 样本不一致——接近人类 evaluator 稳定性
3. 在长篇 TTS 评测中被用作 [[Expressive Richness]] / [[Expressive Hierarchy]] 的主 evaluator
4. **闭源 API 风险**：模型升级后过去评分失去可比性（这是 [[SwanBench-Speech]] §H 自承的 reproducibility 风险）
5. 开源 LALM ([[Qwen3-Omni]] Flash / Instruct) 在同任务上已**超越 GPT-4o**，与 Gemini-2.5-Pro 差距很小

## 代表工作

- [[SwanBench-Speech]] 长篇 TTS 评测主 LALM judge
- Google DeepMind 自家 demo

## 相关概念

- [[Gemini 2.5 Flash]]
- [[Qwen3-Omni]]
- [[GPT-4o]]
- [[Expressive Richness]] / [[Expressive Hierarchy]]
- [[StepFun-Audio-R1]]
