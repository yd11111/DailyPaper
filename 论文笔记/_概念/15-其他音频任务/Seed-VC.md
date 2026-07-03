---
type: concept
aliases: [SeedVC]
---

# Seed-VC

## 定义
Seed-VC 是字节跳动提出的零样本语音转换（Voice Conversion）模型，属于 Seed 系列。核心特点是使用外部生成模型对源音频做扰动，以解决语义 token 中的音色泄漏（timbre leakage）问题，使 [[Flow Matching]] 解码器能更好地从参考 prompt 学习目标音色。

## 核心要点
1. 解决 VC 中语义 token 残留源说话人声学信息的问题
2. 使用外部生成模型扰动源音频，破坏残余声学线索
3. 与 [[Voice Conversion]] 中的属性解耦方法同类

## 代表工作
- Seed-VC (ByteDance, 2024): 原始工作
- [[UnifiedGuidanceFM]]: 引用 Seed-VC 作为解决音色泄漏的相关工作，并提出替代方案（DG 双阶段异构扰动）

## 相关概念
- [[Voice Conversion]]
- [[Flow Matching]]
- [[CosyVoice2]]
