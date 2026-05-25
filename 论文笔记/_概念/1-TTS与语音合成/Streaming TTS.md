---
type: concept
aliases: [流式TTS, 流式语音合成, Streaming Speech Synthesis]
---

# Streaming TTS

## 定义
流式语音合成，指在文本输入尚未完全接收完毕时即开始输出音频的 TTS 模式。核心指标为首包延迟（first-packet latency）和 RTF。

## 核心要点
1. 关键指标: 首包延迟（目标 <200ms）和 RTF（<1 才能实时）
2. 实现方式: chunk-based 处理、causal attention、流式 LM token 交错
3. 挑战: 流式模式因上下文受限可能降低合成质量，尤其在困难文本上
4. 应用场景: 语音对话（如 GPT-4o）、实时翻译、智能客服

## 代表工作
- [[CosyVoice2]]: chunk-aware causal flow matching 实现统一流式/非流式，首包延迟 150ms
- [[Moshi]]: 全双工实时语音对话，160ms 端到端延迟

## 相关概念
- [[Flow Matching]]
- [[CosyVoice]]
