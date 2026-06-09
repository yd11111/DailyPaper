---
type: concept
aliases: [Fish Audio, Fish Audio S2, Fish Speech, FishSpeech]
---

# Fish-Speech

## 定义

开源社区 Fish Audio 开发的自回归 token TTS 系统。Fish Audio S2（4B 参数）是其最新版本，在 [[Seed-TTS-Eval]] 上取得 EN WER 0.99 / ZH CER 0.54 的领先结果。

## 核心要点

1. 自回归离散 token TTS 路线
2. Fish Audio S2 为 4B 参数，多语种支持
3. 在 intelligibility（WER/CER）上是开源 TTS 中最强之一
4. 在低资源东南亚语种（Khmer/Lao/Burmese）上覆盖不足

## 评测/常见数字

- Seed-TTS-Eval EN WER: 0.99（开源最低之一）
- Seed-TTS-Eval ZH CER: 0.54（开源最低之一）

## 代表工作

- [[VoxCPM2]]: 对比基线
- [[Qwen3-TTS]]: 同级竞品

## 相关概念

- [[VALL-E]]
- [[Discrete Audio Token]]
