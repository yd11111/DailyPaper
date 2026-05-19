---
type: concept
aliases: [MiniCPM-o, MiniCPM-o 4.5, MiniCPM-o 2.6]
---

# MiniCPM-o

## 定义

OpenBMB / 清华 NLP 组的端侧多模态大模型系列。以 9B 参数规模追求 **vision + speech + text** 三模态统一理解与生成，重点是 **edge deployable** (<12 GB RAM) 的实时全双工交互。基于 [[MiniCPM-V]] 视觉系列扩展加入语音 encoder/decoder。

## 系列演进

| 版本 | 关键能力 | 关键技术 |
|---|---|---|
| MiniCPM-o 2.6 | 三模态 turn-based | 沿用 MiniCPM-V 视觉 + Whisper + speech decoder |
| **MiniCPM-o 4.5** | **首个端侧实时全双工** | [[Omni-Flow]] + [[TAIL]] + [[llama.cpp-omni]] + [[RLAIF-V]] |

## 架构核心（4.5 版）

- **视觉**: [[LLaVA-UHD]] + [[SigLIP ViT]] (0.4B) + [[Resampler]] 16× 压缩
- **音频**: [[Whisper]] Medium encoder (0.3B) + 5× MLP 时间压缩 → 10 audio tokens/s
- **Backbone**: [[Qwen3-8B]] (8.2B)，仅在文本域 3-4 step/s 解码
- **Speech Decoder**: 0.3B Llama-style，输出 [[CosyVoice|S3 Token]]，25 frames/s
- **Vocoder**: streaming [[Flow Matching]] decoder
- **总参数**: 9.34B，bfloat16

## 核心创新

1. [[Omni-Flow]] 全双工统一序列化框架（chunk=1.0 s）
2. [[TAIL]] 时延感知文-语交错
3. Smooth length reward + [[GRPO]]
4. [[llama.cpp-omni]] 自研端侧推理框架（INT4 RTF 0.20-0.21）

## 代表工作

- [[MiniCPM-o 4.5]]: 当前主线版本（2026/04）
- [[MiniCPM-V]] 4.5: 视觉子系列基础

## 评测/常见数字

- OpenCompass 77.6（同 scale 开源 SOTA、逼近 [[Gemini 2.5 Flash]]）
- SeedTTS Test-ZH CER 0.86 / Test-EN WER 2.38
- RTX 4090 INT4: 212 tok/s, 11 GB, 首 token 0.58 s
- 端侧 <12 GB RAM 实时 full-duplex

## 相关概念

- [[Omni-Flow]]
- [[TAIL]]
- [[MiniCPM-V]]
- [[Qwen3-Omni]]: 同期 30B 对手
- [[Moshi]]: 全双工另一思路
- [[Mini-Omni]]: backbone 直出 speech token 范式
