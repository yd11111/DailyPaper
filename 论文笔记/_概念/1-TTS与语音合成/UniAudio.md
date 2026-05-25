---
type: concept
aliases: []
---

# UniAudio

## 定义
统一音频生成框架，支持 TTS、音频事件生成、音乐生成等多任务。使用 EnCodec token 和 phoneme 输入的 LLM-based 架构。

## 核心要点
1. 多任务统一框架（TTS + Audio + Music）
2. 使用 EnCodec token 作为语音表示
3. CosyVoice 在 TTS 任务上显著优于 UniAudio

## 评测/常见数字
- LibriTTS test-clean WER: 8.74%, SS: 47.56（vs CosyVoice 的 3.17%, 69.49）

## 代表工作
- [[CosyVoice]]: 对比基线

## 相关概念
- [[VALL-E]]
- [[EnCodec]]
