---
type: concept
aliases: [Perceiver AR]
---

# Perceiver AR

## 定义
基于 Perceiver 架构的自回归模型，能处理超长序列输入。在音频领域被用于直接在高码率 SoundStream token 上做自回归建模。

## 核心要点
1. 通过 cross-attention 压缩长输入序列，降低自注意力的二次复杂度
2. 在钢琴音乐生成上音质好（高码率 SoundStream token），但时域结构连贯性不足
3. AudioLM 的 acoustic-only baseline 对标

## 代表工作
- [[AudioLM]]: 指出 Perceiver AR 在高码率 acoustic token 上虽音质好但缺乏长程结构

## 相关概念
- [[SoundStream]]
- [[Acoustic Token]]
- [[Autoregressive]]
