---
type: concept
aliases: [SRMR, Speech-to-Reverberation Modulation Energy Ratio]
domain: TTS
tags: [evaluation, reverb, signal-processing, long-form-tts]
related_maps:
  - "[[TTS-评测体系]]"
created: 2026-05-29
last_updated: 2026-05-29
---

# SRMR (Speech-to-Reverberation Modulation Energy Ratio)

## 定义

经典的非参考混响测量指标，量化语音信号中"直达声"vs"混响成分"的能量比。数值越高表示混响越少（更"干"），越低表示混响越重。

## 核心要点

1. 经典 single-number metric，原本用于评估去混响 / 远场 ASR 输入质量
2. SRMRpy 开源实现：<https://github.com/jfsantos/SRMRpy>
3. 在 TTS 长篇评测中**通常作为标准差使用而非绝对值**——例如 [[SwanBench-Speech]] 用 SRMR 序列的 std 作为 Reverb Consistency 指标（详见 [[SwanBench-Speech]] §3.4 + §C.2），lower std = 更稳定的声学环境
4. 计算时通常配合 VAD 过滤静音窗口避免污染（[[SwanBench-Speech]] 用 FSMN-VAD + Silero-VAD，过滤 > 60% 静音的窗口）

## 代表工作

- [[SwanBench-Speech]] 用 SRMR std dev 作为 Reverb Consistency 评测核心指标，window 3s / stride 2s
- 经典去混响、远场 ASR 评估文献

## 相关概念

- [[Reverb Consistency]]
- [[SwanBench-Speech]]
- [[ZipEnhancer]]（去混响工具）
