---
type: concept
aliases: [Voices-in-the-Wild-Bench, VitW-Bench]
---

# Voices-in-the-Wild-Bench

## 定义

[[MegaASR]] 论文配套的鲁棒 ASR 评测集，共 5,000 条 EN/ZH 双语样本：3,500 条仿真 + 1,500 条真录（互联网采集 + 16 名志愿者）。按 [[Voices-in-the-Wild-2M]] 的 7 种原子场景 + 1 种 Mixed（混合）共 8 类细分。

## 核心要点

1. 同时含真录和仿真，可对比仿真↔真实场景一致性。
2. 8 类细分场景：Noise / Far / Obstructed / Echo / Record / Elc.Dis / Trans / Mixed。
3. Mixed 类是论文专门考察"组合泛化"的关键集合。

## 代表工作

- [[MegaASR]]：发布该 benchmark

## 评测/常见数字

- 规模：5,000 条；语种 EN / ZH
- [[Qwen3-ASR]] 平均 [[WER]]：Mixed Real/Sim 3.30 / 5.39
- [[MegaASR]] Mixed Real/Sim 2.73 / 4.57

## 相关概念

- [[Voices-in-the-Wild-2M]]
- [[MegaASR]]
- [[ASR]]
