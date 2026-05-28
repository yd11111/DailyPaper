---
type: concept
aliases: [MUltiple Stimuli with Hidden Reference and Anchor]
---

# MUSHRA

## 定义

MUltiple Stimuli with Hidden Reference and Anchor，ITU-R BS.1534 标准定义的音频质量主观评测方法。评测者在同一屏幕上对多个系统的音频样本打 0–100 分，其中隐藏了参考音频和低锚点。比 [[MOS]] 更精细，适合 codec 和 vocoder 的质量对比。

## 核心要点

1. 0–100 连续评分，比 MOS 的 1–5 离散量表区分度更高
2. 隐藏参考 + 低锚点设计，校准评测者的评分范围
3. 广泛用于语音 codec 评测（如 [[DAC]]、[[CFMDCTCodec]]、[[FlowDec]] 等）
4. 缺点：需要同时播放多个样本，评测成本高于 MOS

## 代表工作

- ITU-R BS.1534-3 (2015)
