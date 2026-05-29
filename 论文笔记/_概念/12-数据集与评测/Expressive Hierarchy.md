---
type: concept
aliases: [Expressive Hierarchy, 表达层次性, Paragraph-level Expressiveness]
domain: TTS
tags: [evaluation, expressiveness, lalm-as-judge, long-form-tts, paragraph-level]
related_maps:
  - "[[TTS-评测体系]]"
created: 2026-05-29
last_updated: 2026-05-29
---

# Expressive Hierarchy

## 定义

衡量长篇合成音频**段落级（paragraph-level）表达动态变化**的指标——超出单 segment 内的平均强度，关注 emotional variation / vocal dynamics / scene appropriateness 是否随叙事弧线展开。

## 核心要点

1. **计算方法**（[[SwanBench-Speech]] §C.7）：**整段（不切片）**喂给 LALM（论文用 [[Gemini 3 Pro]]）→ 评 1-5 分
2. **为何不切片**：切片会破坏 narrative flow 的整体感——只有看整段才能判断"激动该升 / 平静该降"的层次是否合理
3. **人工对齐**（[[SwanBench-Speech]] §D.4）：Gemini-3-Pro SRCC=0.62（相比 Richness 的 0.71 略低）——意味着排名可信但分数细微差异需谨慎
4. **真人参考**：[[SwanBench-Speech]] Tab.2 真人长篇 Hierarchy = 3.94；最强模型（[[Gemini-2.5-pro-tts]]）3.51；**所有模型距真人差距均 ≥ 0.4**
5. **与 [[Expressive Richness]] 的区别**：Richness 关注"平均够不够"，Hierarchy 关注"动态变化是否合理"——一个 NAR 模型如果全程都用"中等表达力"输出，可能 Richness 还行但 Hierarchy 必差
6. **AR vs NAR 暴露在此指标上**：[[F5-TTS]]（NAR）Hierarchy 2.77，[[ZipVoice-Dialog]] (NAR) 2.80，明显低于 AR 同类——这是 [[SwanBench-Speech]] §5.2 推出"NAR 表达性平淡"判断的核心证据

## 代表工作

- [[SwanBench-Speech]] 提出，作为长篇区别于句子级的核心新维度
- 与 [[Expressive Richness]] 一起构成 Expressiveness Challenge 的双指标

## 相关概念

- [[Expressive Richness]]
- [[Gemini 3 Pro]]
- [[Prosodic Coherence]]
- [[SwanBench-Speech]]
- [[F5-TTS]] / [[ZipVoice]]（NAR 弱在此指标的代表）
