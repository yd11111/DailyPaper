---
type: concept
aliases: [Voices-in-the-Wild, VitW-2M, VitW2M]
---

# Voices-in-the-Wild-2M

## 定义

[[MegaASR]] 论文提出的大规模鲁棒 ASR 训练数据集，规模约 240 万条音频、约 11k 小时。通过谱图级 code-based 仿真，覆盖 **7 种原子声学效应** × 由 agent 校验的 **54 种物理可信复合场景**。

## 核心要点

1. 7 种原子声学效应：noise / far-field / obstructed / echo & reverb / recording / electronic distortion / transmission dropout。
2. 54 种复合场景：从 7 原子中选 2–5 个组合，由 agent 校验物理合理性。
3. 干净语音源：[[LibriSpeech]]、[[Common Voice]]、[[WenetSpeech]]、[[AISHELL-1]]；噪声源：[[MUSAN]]、DNS Challenge、ESC-50、UrbanSound8K。
4. 难度参数 $k \in [0,1]$，采用 Linear 采样；过滤掉 [[WER]] > 70% 的不可学习样本。
5. [[Qwen3-ASR]] 在其上平均 [[WER]] 18.42%，反映数据难度。

## 代表工作

- [[MegaASR]]：发布并使用该数据集训练

## 评测/常见数字

- 规模：2.4M 条 / ~11k h
- baseline [[WER]]: 18.42%（[[Qwen3-ASR]]）

## 相关概念

- [[Voices-in-the-Wild-Bench]]
- [[CHiME-4]]
- [[VOiCES]]
- [[NOIZEUS]]
- [[MUSAN]]
