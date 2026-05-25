---
type: concept
aliases: [Grapheme-to-Phoneme, 字素到音素转换]
---

# G2P

## 定义
Grapheme-to-Phoneme 转换，将文字（grapheme）转换为音素（phoneme）序列的前端模块。传统 TTS 流水线的标准组件。

## 核心要点
1. 处理多音字、异读词、缩写等文本规范化问题
2. 常用工具: Montreal Forced Aligner (MFA)、pypinyin（中文）、Phonemizer
3. 近年趋势: 部分模型（如 CosyVoice 2、E2 TTS、F5-TTS）直接使用 BPE 文本 token，跳过 G2P 步骤

## 代表工作
- [[CosyVoice2]]: 移除 G2P，直接使用 BPE tokenizer 处理原始文本
- [[F5-TTS]]: 无 phoneme、无 duration 的纯 flow matching 方案

## 相关概念
- [[Duration Predictor]]
- [[Forced Alignment]]
