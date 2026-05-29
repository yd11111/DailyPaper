---
type: concept
aliases: [Expressive Richness, 表达丰富度, Sentence-level Expressiveness]
domain: TTS
tags: [evaluation, expressiveness, lalm-as-judge, long-form-tts]
related_maps:
  - "[[TTS-评测体系]]"
created: 2026-05-29
last_updated: 2026-05-29
---

# Expressive Richness

## 定义

衡量长篇合成音频**段内（segment-level）平均表达力**的指标，关注 emotional resonance / character portrayal / storytelling 三个维度。本质上类似句子级表达度评测的扩展版。

## 核心要点

1. **计算方法**（[[SwanBench-Speech]] §C.6 + Eq.4）：把音频切成 **10s 互不重叠**段 $\{c_i\}_{i=1}^M$ → 每段用 LALM（论文用 [[Gemini 3 Pro]]）评 1-5 分 → 取算术平均
2. **10s window 设计选择**：对齐 chunk-based 长篇合成 pipeline 的典型生成长度，避免 inter-chunk 不一致干扰
3. **人工对齐**（[[SwanBench-Speech]] §D.4）：Gemini-3-Pro SRCC=0.71（200 样本 vs 10-rater MOS）；**传统 MOS 网络（UTMOS/UTMOSv2/SQUIM-MOS/DNSMOS）与人工几乎不相关**——说明 MOS 训练集普遍缺乏 expressive label
4. **真人参考**：[[SwanBench-Speech]] Tab.2 真人长篇 Richness = 4.35；当前最强模型（[[Gemini-2.5-pro-tts]]）4.14；**这是当前长篇 TTS 与真人最大的 gap**
5. **与 [[Expressive Hierarchy]] 的区别**：Richness 是 segment 内平均强度，Hierarchy 是 paragraph 级动态变化能力

## 代表工作

- [[SwanBench-Speech]] 提出并作为 Expressive 维度主指标
- 启发自 [[EmergentTTS-Eval]]（Manku et al. 2025）的 LALM 评测方法

## 相关概念

- [[Expressive Hierarchy]]
- [[Gemini 3 Pro]]
- [[EmergentTTS-Eval]]
- [[MOS]]
- [[LALM-as-judge]]
- [[SwanBench-Speech]]
