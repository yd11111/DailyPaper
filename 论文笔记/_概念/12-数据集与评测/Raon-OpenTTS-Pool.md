---
type: concept
aliases: [Raon Pool]
---

# Raon-OpenTTS-Pool

## 定义

由 [[Raon-OpenTTS]] 团队（KRAFTON）发布的全开源英文 TTS 训练数据池，**615K 小时、240M 段**语音，由 11 个公开数据集 + 自建预处理的 [[Raon-YouTube-Commons]]（335K h）聚合而成。

## 核心要点

1. **规模**: 615K 小时 = 当前最大的「完全开源 + 可商用为主」英文 TTS 训练数据
2. **入池规则**: 英文、单数据集 ≥500 h、单段 <30 s
3. **统一格式**: 16 kHz mono、64 kbps Opus
4. **来源**: Raon-YouTube-Commons (335K h) + Emilia-YODAS (92K h) + [[Emilia]] (47K h) + LibriHeavy (42K h) + [[HiFiTTS2]] (37K h) + People's Speech (28K h) + [[VoxPopuli]] (17K h) + GigaSpeech (10K h) + SPGISpeech (5K h) + SPGISpeech 2.0 (889 h) + [[LibriTTS|LibriTTS-R]] (552 h)
5. **License**: 大部分 CC BY 4.0 / Public Domain / Apache 2.0；Emilia-YODAS 为 CC BY-NC

## 整体质量统计

| 指标 | 平均值 |
|---|---|
| [[DNSMOS]] | 2.83 |
| [[Whisper]] WER | 0.24 |
| [[Speech Ratio]] | 0.89 |

## 代表工作

- [[Raon-OpenTTS]]: 本数据池的发布论文与基线模型

## 相关概念

- [[Raon-OpenTTS-Core]]: 本池 15% 组合过滤后的训练子集
- [[Emilia]]、[[LibriTTS]]、[[HiFiTTS2]]、[[VoxPopuli]]: 组成来源
- [[Raon-YouTube-Commons]]: 占比最大的自建子集
