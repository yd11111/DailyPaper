---
type: concept
aliases: [CosyVoice 2, 阿里 CosyVoice]
---

# CosyVoice

## 定义

阿里达摩院开源的多语种零样本 TTS 系列。CosyVoice 1/2 都基于 supervised semantic token + flow matching + vocoder 的三段式：先用 ASR 自监督训出 semantic speech token，再让 LLM 自回归预测这些 token，最后用 [[Flow Matching]] 把 token 还原成 Mel，HiFi-GAN 类声码器解码到波形。CosyVoice 2 把帧率压到 25 Hz、支持流式合成。

## 数学形式

二段式条件概率：
$$
p(s | \text{text}, \text{prompt}) = p_{\text{LLM}}(\text{token} | \text{text}, \text{prompt}) \cdot p_{\text{Flow}}(\text{Mel} | \text{token})
$$

帧率：CosyVoice 2 = 25 Hz；输入 24 kHz waveform。

## 核心要点

1. **Supervised semantic token**：用 ASR loss 监督 codec encoder，让 token 自带语义信息（不是纯重建驱动）
2. **Flow Matching 解码器**：把 token 解码到 Mel，比扩散更快
3. **Streaming TTS**：CosyVoice 2 把整条流水线做了 chunk-based streaming
4. 多语种 + 跨语言克隆能力强，是 2024-2025 中文社区零样本 TTS 主流基线

## 代表工作

- [[VibeVoice]]: 在 SEED 短句基准上对比 CosyVoice 2 (帧率 25 Hz vs VibeVoice 7.5 Hz)
- CosyVoice 2 原论文 (arXiv 2412.10117)
- [[OmniFlatten]]: 复用 CosyVoice 的 speech tokenizer（单码本 4096 codes）+ OT-CFM detokenizer + HiFi-GAN，作为全双工对话模型的语音 I/O 组件

## 评测/常见数字

- SEED test-zh CER: 1.45 (CosyVoice 2)
- SEED test-en WER: 2.57 (CosyVoice 2)
- 帧率 25 Hz；与 [[Spark-TTS]] (50 Hz) 相比快约 2×

## 相关概念

- [[Flow Matching]]
- [[Audio Codec]]
- [[F5-TTS]]
