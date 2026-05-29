---
type: concept
aliases: [Timbre Consistency, 音色一致性, Within-Utterance Timbre]
domain: TTS
tags: [evaluation, speaker, long-form-tts, within-utterance]
related_maps:
  - "[[TTS-评测体系]]"
created: 2026-05-29
last_updated: 2026-05-29
---

# Timbre Consistency

## 定义

针对**单段长篇合成音频内部**的音色稳定性度量——区别于经典 SIM-O / SECS（衡量与 prompt 的相似度），这里衡量"音频自己内部是否音色漂移"。

## 核心要点

1. **计算方法**（[[SwanBench-Speech]] §C.1）：3s window / 2s stride 提取 [[WavLM]]-TDCNN speaker embedding 序列 → 所有 distinct pair cosine 平均
2. **多说话人扩展**：先用 forced alignment（[[Paraformer]] 中 / [[WhisperX]] 英）分流 → 每个 speaker 内部按上式 → 跨说话人平均
3. **人工对齐**：[[SwanBench-Speech]] §D.1 报告 SRCC=0.75 / PLCC=0.77 / KRCC=0.59
4. **经验阈值**（[[SwanBench-Speech]] §D.1）：
   - < 0.85：显著音色漂移（多说话人下也可能是切换不准）
   - 0.85 - 0.90：轻微音色 mutation
   - ≥ 0.93：接近真人录音
5. **局限**：基于全局平均，对周期性 / 循环型音色波动可能漏判
6. 与 [[SIM-O]] / [[SECS]] 互补：SIM-O 是"和 prompt 像不像"，Timbre Consistency 是"自己稳不稳定"

## 代表工作

- [[SwanBench-Speech]] 长篇 TTS 评测的核心 Acoustics 指标
- 与传统 zero-shot 评测里的 SIM-O 形成互补维度

## 相关概念

- [[SIM-O]] / [[SECS]] / [[SIM-R]]
- [[WavLM]]
- [[Speaker Encoder]]
- [[Reverb Consistency]]
- [[SwanBench-Speech]]
