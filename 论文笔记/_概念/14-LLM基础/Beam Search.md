---
type: concept
aliases: [束搜索, Beam Decoding]
---

# Beam Search

## 定义
序列生成时的搜索算法，维护 beam_size 个候选序列并行扩展，每步保留概率最高的 top-k 候选。相比贪心解码（beam=1）可找到更优的全局序列，相比穷举搜索计算可行。

## 核心要点
1. beam_size 越大结果越接近最优但计算量线性增长
2. 适用于需要高准确率的 seq2seq 任务（如 ASR、text->semantic token）
3. 不适用于需要多样性的生成任务（如 TTS 中的声学 token 生成），此时用温度采样

## 代表工作
- [[SPEAR-TTS]]: S1 推理用 beam=10 的 beam search 保证文本可懂度
- [[Whisper]]: ASR 解码常用 beam search

## 相关概念
- [[Autoregressive]]
- [[Transformer]]
