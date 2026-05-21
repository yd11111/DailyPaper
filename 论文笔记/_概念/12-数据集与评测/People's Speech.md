---
type: concept
aliases: [Peoples Speech, MLCommons People's Speech]
---

# People's Speech

## 定义

由 **MLCommons** 发布的大规模开源英文 ASR 数据集，约 **30K 小时**（[[Raon-OpenTTS]] 统计 28K h），来源包含政府听证、播客、视频字幕等。CC BY 4.0 授权，可商用。

## 核心要点

1. **规模**: 30K h，是开源英文 ASR 中规模较大的一员
2. **分 Clean / Dirty 两子集**: Clean 子集 [[Raon-OpenTTS]] 过滤保留率 83.9%，Dirty 子集只保留 48.2%
3. **质量**: [[DNSMOS]] 2.63（偏低，因含大量 in-the-wild 录音）、[[Whisper]] WER 0.25
4. **License**: CC BY 4.0

## 代表工作

- People's Speech 数据集论文（Galvez et al. 2021）
- [[Raon-OpenTTS]]: 作为 Pool 来源之一

## 相关概念

- [[GigaSpeech]] / [[VoxPopuli]]: 同档可商用 ASR 数据
- [[Raon-OpenTTS-Pool]]
