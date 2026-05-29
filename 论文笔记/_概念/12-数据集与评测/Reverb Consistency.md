---
type: concept
aliases: [Reverb Consistency, 混响一致性, Acoustic Field Consistency]
domain: TTS
tags: [evaluation, reverb, long-form-tts, acoustic-environment]
related_maps:
  - "[[TTS-评测体系]]"
created: 2026-05-29
last_updated: 2026-05-29
---

# Reverb Consistency

## 定义

长篇合成音频在时间维度上的**声学环境稳定性**度量。对单段音频计算 [[SRMR]] 序列后取**标准差**——值越低表示混响越稳定（不漂移）。

## 核心要点

1. **计算方法**（[[SwanBench-Speech]] §C.2）：3s window / 2s stride 计算 [[SRMR]] → VAD 过滤 > 60% 静音的窗口 → 取序列 std
2. **方向**：↓（std 越低越稳定）
3. **真人参考**：[[SwanBench-Speech]] Tab.2 实测真人单说话人 SRMR std ≈ 1.91，对话场景真人 SRMR std ≈ 2.73；合成系统多在 1.4 - 3.6 之间
4. **反直觉发现**：[[Minimax-Speech 02 HD]] (1.38) 和 [[Gemini-2.5-pro]] (1.44) 比真人录音 (1.91) **更稳定**——这可能说明合成系统过于"干净/单一"反而失去真实感
5. **场景依赖**：[[SwanBench-Speech]] §C.2 自承 Outdoor Live Streaming 等场景**本就需要 dynamic acoustic shifts**，"std 低 = 好"的隐含假设在这些场景下违反真实感
6. **对话场景挑战**：频繁说话人切换打断混响连续性，所有合成系统在对话 Reverb 上都落后真人（[[SwanBench-Speech]] 闭源平均 3.36 vs 真人 2.73）

## 代表工作

- [[SwanBench-Speech]] 首次把 SRMR std dev 作为长篇 TTS 主要 acoustic metric
- 提示长篇/对话 TTS 的核心难题之一：**全局声学场一致性**

## 相关概念

- [[SRMR]]
- [[Acoustic Environment]]
- [[Timbre Consistency]]（另一个 within-utterance 一致性指标）
- [[SwanBench-Speech]]
- [[ZipEnhancer]]（去混响工具）
