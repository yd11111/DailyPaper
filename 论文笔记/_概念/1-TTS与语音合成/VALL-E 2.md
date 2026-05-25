---
type: concept
aliases: [VALLE2, VALL-E2]
---

# VALL-E 2

## 定义

微软提出的 VALL-E 升级版，通过 Repetition Aware Sampling 和 Grouped Code Modeling 解决了原始 VALL-E 的鲁棒性问题，号称首个在 LibriSpeech / VCTK 上达到人类水平的零样本 TTS 系统。

## 核心要点

1. **Repetition Aware Sampling**: 检测 AR 解码中的 token 重复，动态切换 nucleus sampling 和 random sampling，解决 infinite loop 和重复问题
2. **Grouped Code Modeling**: 将多层 RVQ code 分组后一次性预测，减少 AR 步数
3. 在 LibriSpeech 和 VCTK 上 SMOS / CMOS 首次达到人类水平

## 代表工作

- [[VALL-E]]: 前作
- VALL-E 2 (Chen et al., 2024): 本工作

## 评测/常见数字

- LibriSpeech: MOS 达到人类水平（无统计显著差异）
- 解决了 VALL-E 的鲁棒性问题（跳词/重复显著减少）

## 相关概念

- [[VALL-E]]
- [[RVQ]]
- [[Zero-shot TTS]]
- [[Autoregressive Model]]
