---
type: concept
aliases: [编解码合并, Codec-Merging, Merged Codec]
domain: Codec
tags: [codec, downsampling, inference-efficiency, rvq]
created: 2026-05-29
last_updated: 2026-05-29
related_maps:
  - "[[TTS-表示层地图]]"
---

# Codec Merging（编解码合并）

## 定义

一种**无需重训 codec** 的推理期下采样手段：在神经 codec（如 [[EnCodec]]）的某层 [[RVQ]] 量化前插入 merge 模块，降低该层离散码的时间分辨率（如第一层从 75Hz 降到 37.5Hz），从而缩短下游 codec 语言模型的自回归序列、加速推理。

## 数学形式

设第 $d$ 层合并率为 $m_d$，残差输入 $r_d^{F\times T}$：
1. **下采样**：average pooling 到 $r_{m_d}^{F\times(T/m_d)}$
2. **上采样**：repeat 回原长 $T$
3. 经 merge 的残差送入 VQ，最近邻查表量化为 $C_d^{1\times T}$

因连续 $m_d$ 帧被强制相同，有效分辨率下降 $m_d$ 倍。

## 核心要点

1. **不动 codec 权重**，纯推理期操作，迁移成本低。
2. 实践表明**只合并第一层**（$m{=}2$）几乎不损重建质量（PESQ 3.62→3.57）；合并更多层（1-4、1-8）质量显著下降且不再加速。
3. 第一层 3×/4× 下采样仅轻微掉质量，存在进一步提速空间。
4. 与自回归步数线性相关；又因 self-attention 复杂度随序列指数增长，减半采样率可带来 >2× 加速。

## 代表工作

- [[VALL-E R]]: 提出 codec-merging，把 EnCodec 第一层 2× 下采样，推理时间缩减 60%+

## 评测/常见数字

- EnCodec 8 层 RVQ / 24kHz / 75Hz；merge 第一层后第一层 37.5Hz
- 只合第一层 2×：PESQ(NB) 3.57 / STOI 0.947（vs 不合并 3.62 / 0.950）

## 相关概念

- [[EnCodec]] / [[RVQ]]
- [[Acoustic Token]]
