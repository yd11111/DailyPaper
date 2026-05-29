---
type: concept
aliases: [RAS, 重复感知采样]
---

# Repetition Aware Sampling

## 定义

[[VALL-E 2]] 提出的推理期采样策略：在 AR codec LM 解码时，先用 nucleus sampling 取候选 token，再计算它在前序窗口内的局部重复率；若重复率超阈值则改用 random sampling 重采。目的是兼得 nucleus sampling 的稳定性与 random sampling 规避无限循环的能力。纯推理期 trick，不改训练、几乎不增延迟。

## 数学形式

重复率（窗口 $K$）：

$$
r \leftarrow \frac{1}{K}\sum_{k=0}^{K}\mathbb{1}_{c_{t'}=c_{t'-k}}
$$

若 $r > t_r$（重复阈值）则对该 code 改用 random sampling。VALL-E 2 取 $K=10$、$t_r=0.1$、top-p $v\in[0,0.8]$。

## 核心要点

1. 解决小 top-p nucleus sampling 的无限循环问题，同时保留其稳定性
2. 关键效果：让模型能在极小 top-p（甚至 0）下稳定解码，从而拿到很低（甚至低于 GT）的 WER
3. 几乎零成本：采样开销相对模型推理可忽略，不增延迟
4. 通用：可迁移到任意 codec-LM TTS 的 AR 解码

## 代表工作

- [[VALL-E 2]] (Chen et al., 2024): 提出该方法

## 评测/常见数字

- LibriSpeech ref utterance, single sampling: WER 由 VALL-E 的 3.1 降到 1.5
- VCTK single sampling: WER 大致砍半

## 相关概念

- [[VALL-E 2]]
- [[Autoregressive Model]]
- [[Zero-shot TTS]]
