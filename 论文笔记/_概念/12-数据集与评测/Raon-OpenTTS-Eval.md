---
type: concept
aliases: [Raon Eval, Raon-Eval]
---

# Raon-OpenTTS-Eval

## 定义

由 [[Raon-OpenTTS]] 团队发布的 **TTS 鲁棒性 benchmark**，6,000 条 prompt-text 对、12 个数据集，覆盖 **Clean / Noisy / Wild / Expressive** 四种声学场景，弥补 [[Seed-TTS-eval]] / CV3 只测干净朗读音频的盲区。

## 四个场景

| 场景 | 段数 | 数据集 |
|---|---|---|
| **Clean** | 2,500 | [[LibriSpeech]]-clean、ST American English、CMU-ARCTIC、L2-ARCTIC、VCTK |
| **Noisy** | 1,000 | [[LibriSpeech]]-other、[[TED-LIUM|TED-LIUM 3]] |
| **Wild** | 1,000 | AMI-IHM、AMI-SDM（多人远场会议） |
| **Expressive** | 1,500 | Expresso、CREMA-D、EmoV-DB |

## 核心要点

1. **Wild 场景是关键过滤器**: [[F5-TTS]] 在 AMI 上 WER 高达 **136%**（hallucination 严重），[[Qwen3-TTS]] 79%，[[VoxCPM]] 44%；[[Raon-OpenTTS]]-1B 仅 5.61%
2. **客观指标**: WER（[[Whisper]] 反解）+ SIM（speaker embedding）+ [[DNSMOS]]
3. **主观指标**: [[CMOS]]（自然度）+ [[SMOS]]（说话人相似度），Amazon MTurk 30 item × 6 annotator/condition
4. **设计动机**: 让 TTS 评测体系跳出 audiobook / studio 录音的舒适区

## 代表工作

- [[Raon-OpenTTS]]: 发布与首批结果

## 相关概念

- [[Seed-TTS-eval]]: 同样是 zero-shot TTS 评测集，但只覆盖干净场景
- [[Zero-shot TTS]]: 评测范式
- [[CMOS]]、[[SMOS]]: 主观评测协议
