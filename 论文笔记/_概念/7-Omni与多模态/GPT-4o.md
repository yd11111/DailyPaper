---
type: concept
aliases: [GPT-4o, GPT4o]
---

# GPT-4o

## 定义
OpenAI 发布的多模态基础模型，原生支持文本、音频、图像输入输出，被认为是首个工业级多模态 omni 模型。

## 核心要点
1. 支持 speech-to-speech 对话，Realtime API 报告 232/320 ms 音频响应延迟
2. 实际 API TTFB ~500 ms，目标 voice-to-voice ~800 ms
3. 音频输出具有情感表达、笑声等副语言特征
4. 未公开架构细节

## 评测/常见数字
- 官方 audio response: 232/320 ms
- API TTFB: ~500 ms
- 目标 voice-to-voice: ~800 ms

## 代表工作
- OpenAI (2024)

## 相关概念
- [[Wan-Streamer]]: 延迟对比基线
- [[Moshi]]: 全双工语音对话对标
- [[Full-Duplex]]
