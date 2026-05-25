---
type: concept
aliases: []
---

# Voicebox

## 定义
Meta 提出的非自回归 TTS 系统，基于 Flow Matching 的 mask-and-infill 范式。给定部分音频和完整文本，通过条件流匹配生成缺失部分，支持 zero-shot TTS、语音编辑、风格迁移等。

## 核心要点
1. 非自回归 Flow Matching 生成，需要 phoneme duration 和 forced alignment
2. mask-and-infill 范式：训练时随机 mask 一段音频，推理时 mask 目标位置
3. 是 F5-TTS、CosyVoice CFM 模块等后续工作的重要参考
4. CosyVoice 的 CFM 不需要 phoneme 和 duration（对比优势）

## 代表工作
- [[F5-TTS]]: 受 Voicebox 启发但去掉了 phoneme 依赖
- [[CosyVoice]]: CFM 部分受 Voicebox 影响

## 相关概念
- [[Conditional Flow Matching]]
- [[Flow Matching]]
- [[Forced Alignment]]
- [[F5-TTS]]
