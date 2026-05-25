---
type: concept
aliases: [Seed-TTS, SeedTTS-eval, Seed-TTS-eval]
---

# SeedTTS

## 定义

字节跳动提出的大规模 zero-shot TTS 系统（Anastassiou et al., 2024）。同时提供了 SeedTTS-eval 评测基准，包含 test-en（1000 条，来自 Common Voice）和 test-zh（2000 条，来自 DiDiSpeech）两个子集，是 zero-shot TTS 领域广泛使用的标准评测集。

## 核心要点

1. SeedTTS-eval 是 zero-shot TTS 的主流评测基准之一
2. 评测维度包括 WER（可懂度）、SS/SIM-O（说话人相似度）、主观 MOS
3. test-en 和 test-zh 分别代表英文和中文评测场景

## 代表工作

- [[IndexTTS2]]: 在 SeedTTS test-en/zh 上取得 SOTA
- [[CosyVoice2]]: 主要对比基准之一

## 相关概念

- [[LibriSpeech]]
- [[AISHELL-1]]
