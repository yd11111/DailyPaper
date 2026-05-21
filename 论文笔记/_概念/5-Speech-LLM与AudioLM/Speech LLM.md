---
type: concept
aliases: [SpeechLM, SLM, 语音大模型]
---

# Speech LLM

## 定义

把音频信号（waveform 或离散音频 token）作为输入或输出模态，与文本 LLM 联合建模的大模型范式。覆盖语音理解（ASR、QA、emotion）、生成（TTS）、对话（duplex chat）等任务。

## 核心要点

1. **结构主流路线**：
   - **三段式**（如 [[Qwen3-ASR]]、[[Qwen2.5-Omni]]）：speech encoder → aligner（adapter）→ LLM
   - **离散 token + LLM**（如 [[VALL-E]]、AudioLM、[[Kimi-Audio]]）：先用 [[Audio Codec|codec]] 离散化，再当 token 喂给 LLM
   - **端到端联合**（如 [[Moshi]]）：双流并行建模听+说
2. **关键挑战**：模态对齐、声学锚定（防止 [[ASR Hallucination|幻觉]]）、低延迟流式生成、多说话人混叠处理。
3. **代表评测**：[[LibriSpeech]] / [[Fleurs]] / [[Voices-in-the-Wild-Bench]]（ASR），[[Seed-TTS-eval]]（TTS），AlpacaEval-Voice（对话）。

## 代表工作

- ASR 方向：[[Qwen3-ASR]]、[[Whisper]]、[[Step-Audio-2]]、[[Kimi-Audio]]
- 对话 / Omni：[[Qwen2.5-Omni]]、[[Moshi]]、GPT-4o、GLM-4-Voice
- 鲁棒 ASR：[[MegaASR]]

## 相关概念

- [[Audio Codec]]
- [[ASR]]
- [[TTS]]
- [[Full-Duplex]]
- [[Omni]]
