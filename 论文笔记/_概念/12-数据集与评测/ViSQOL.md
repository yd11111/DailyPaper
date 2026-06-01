---
type: concept
aliases: [Virtual Speech Quality Objective Listener, visqol]
---

# ViSQOL

## 定义

Virtual Speech Quality Objective Listener：开源的客观音频质量评估指标，通过比较参考与退化音频的频谱相似性来预测主观质量分（类似 POLQA/PESQ 但无许可限制）。输出范围 1-5，与 MOS 对齐。

## 核心要点

1. 免费开源替代 PESQ/POLQA，适合学术研究
2. 与主观 MUSHRA 分数高相关
3. SoundStream 用其做模型选择和消融实验的客观指标
4. 适用于语音、音乐、通用音频的质量评估

## 代表工作

- [[SoundStream]]: 首次大规模用 ViSQOL 做 neural codec 评测
- [[EnCodec]]: 沿用 ViSQOL 作为客观指标之一

## 评测/常见数字

- 高质量 codec (6kbps): ViSQOL ~4.0
- 低码率 (3kbps): ViSQOL ~3.7-3.8
- 参考（无损）: 4.5+

## 相关概念

- [[MUSHRA]]
- [[PESQ]]
- [[MOS]]
