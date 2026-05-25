---
type: concept
aliases: [G2P, grapheme-to-phoneme, 字素到音素转换]
---

# Grapheme-to-Phoneme

## 定义
将文字（grapheme）转换为音素（phoneme）序列的模块，是 TTS 前端的核心组件。将自然语言文本映射为发音表示（通常用 IPA 或 CMUDict）。

## 核心要点
1. 传统 TTS（FastSpeech, Tacotron）通常依赖 G2P 作为前端
2. 低资源语言往往缺乏高质量开源 G2P，限制 TTS 发展
3. 部分现代模型（YourTTS, F5-TTS）选择绕过 G2P，直接用 raw text/grapheme 输入
4. 跳过 G2P 可能导致发音错误（尤其多音字、外来词）

## 代表工作
- [[YourTTS]]: 选择不用 G2P，直接 raw text 输入以支持多语言
- [[F5-TTS]]: 无 phoneme、无 duration predictor 的纯 Flow Matching TTS

## 相关概念
- [[VITS]]
- [[Stochastic Duration Predictor]]
