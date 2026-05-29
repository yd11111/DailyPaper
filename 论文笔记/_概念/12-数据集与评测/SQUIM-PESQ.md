---
type: concept
aliases: [SQUIM-PESQ, SQUIM, Reference-Free PESQ]
domain: TTS
tags: [evaluation, fidelity, reference-free, signal-processing]
related_maps:
  - "[[TTS-评测体系]]"
created: 2026-05-29
last_updated: 2026-05-29
---

# SQUIM-PESQ

## 定义

Torchaudio-SQUIM 套件中的 reference-free PESQ 预测模型，输入退化音频直接输出 PESQ 分数估计，无需 reference signal。范围约 -0.5 ~ 4.5，语音通常 > 1.0。

## 核心要点

1. **解决经典 PESQ 必须有 reference 的问题**——TTS 合成场景下没有 ground truth 配对音频，传统 PESQ 不可用
2. 由 Kumar et al. ICASSP 2023（"Torchaudio-SQUIM: reference-less speech quality and intelligibility measures"）提出
3. 实现：通过 Torchaudio 官方接口 `torchaudio.tutorials.squim_tutorial`
4. 与人工 Fidelity MOS 相关性：[[SwanBench-Speech]] §D.2 报告 SRCC=0.72 / PLCC=0.47 / KRCC=0.53（在 50 个中文+长篇样本上验证 generalization）
5. 警告：SQUIM 训练集主要是英文 + 句子级，跨语种 + 长篇泛化能力有限——但实证表现尚可

## 代表工作

- [[SwanBench-Speech]] 用作 Sound Fidelity 评测指标
- Torchaudio 自带 demo

## 相关概念

- [[PESQ]]（参考版 PESQ）
- [[Sound Fidelity]]
- [[DNSMOS]]（另一类 reference-free MOS）
- [[SwanBench-Speech]]
