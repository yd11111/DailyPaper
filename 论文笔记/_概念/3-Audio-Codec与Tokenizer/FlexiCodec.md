---
type: concept
aliases: [Flexi-Codec, 动态帧率codec]
---

# FlexiCodec

## 定义
动态帧率神经音频编解码器，通过 Frame Merging 机制将固定 12.5 Hz 的语义特征自适应合并为变长序列，平均帧率可降至 ~6.25 Hz，同时保持强重建质量。使用 [[FSQ]]（Finite Scalar Quantization）替代传统 [[RVQ]]。

## 核心要点
1. **双流编码**：语义编码分支（基于 [[SenseVoice]] ASR 特征 + Frame Merging + FSQ）和声学编码分支（RVQ）并行，SLM 应用中通常只用语义分支
2. **Frame Merging**：基于相邻帧余弦相似度做贪心合并——相似度 > 阈值 τ 的帧对被平均，产出变长序列 + 帧长度属性
3. **帧率可控**：通过调节合并阈值 τ（0.8~1.0），可控帧率从 ~3 Hz 到 12.5 Hz
4. **FSQ 量化**：单码本，codebook 大小视配置而定（FlexiSLM 中使用 FlexiCodec 预训练版本）
5. **解码器**：NAR [[Flow Matching]] Transformer（363M，VoiceBox 风格）+ [[Vocos]] 声码器输出 24 kHz 波形

## 代表工作
- [[FlexiSLM]]: 首次将 FlexiCodec 集成到端到端 SLM 框架中
- Li et al. 2025b (arXiv:2510.00981): FlexiCodec 原论文，验证于 0.3B TTS pipeline

## 开源
- GitHub: https://github.com/amphionteam/flexicodec

## 相关概念
- [[FSQ]]
- [[RVQ]]
- [[SenseVoice]]
- [[Audio Codec]]
- [[Semantic Token]]
- [[Frame Rate]]
