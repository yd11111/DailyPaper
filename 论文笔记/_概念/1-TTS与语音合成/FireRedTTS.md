---
type: concept
aliases: [FireRed TTS]
---

# FireRedTTS

## 定义

小红书 FireRed Team 推出的工业级 TTS 基础框架（2024），采用语义感知 tokenizer + AR 语言模型 + 两阶段波形生成器架构。语义 token 帧率 40ms（25 Hz），码本 16384。

## 核心要点

1. 完整的数据处理 pipeline：624k 小时原始音频 → 248k 小时标注数据
2. SAST tokenizer：[[HuBERT]] 语义编码器 + [[ECAPA-TDNN]] 声学编码器，解耦内容与音色
3. 400M 参数 decoder-only [[Transformer]] 做 AR 语义 token 生成
4. 双解码路径：[[Flow Matching]] decoder（高质量）+ Streamable decoder（[[Mel Codec]]，低延迟）
5. [[BigVGAN]]-V2 超分辨率声码器输出 48 kHz
6. 指令微调支持 4 类情感 + 13 种副语言行为

## 代表工作

- [[FireRedTTS]]: 原文 (arXiv 2409.03283, Guo et al. 2024)
- [[FireRedTTS-2]]: 后续改进版本
- [[VibeVoice]]: 短句基线对比

## 评测/常见数字

- CoMOS: 4.32（CosyVoice 4.15, GT 4.53）
- 中文句级错误率: 2.09%（CosyVoice 5.68%）
- PUGC few-shot MOS: 4.65, SIM: 78.92%
- 情感控制准确率: 97-100%（instruction tuning 后）
- 帧率: 25 Hz (40ms)

## 相关概念

- [[CosyVoice]]
- [[VALL-E]]
- [[Semantic Token]]
- [[Flow Matching]]
- [[Mel Codec]]
- [[ECAPA-TDNN]]
