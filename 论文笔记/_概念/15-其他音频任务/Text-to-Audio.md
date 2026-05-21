---
type: concept
aliases: [Text-to-Audio, TTA, text2audio, 文本到音频生成]
---

# Text-to-Audio

## 定义

Text-to-Audio (TTA) 是给定文字描述（"狗在叫，背景有车流"）生成对应**通用音频**（音效 / 音乐 / 环境声）的任务。**不等同于** [[TTS]]：TTS 输出语音，TTA 输出任意音频。代表系统包括 [[AudioLDM]] / [[Tango]] / [[Stable Audio]] / [[MovieGen-Audio]]。

## 核心要点

1. **输入**：自然语言描述（多为 1–2 句）
2. **输出**：通常 10 秒左右的 16 kHz / 24 kHz / 48 kHz 音频片段
3. **主流架构**：CLAP 文本编码 + audio VAE latent + diffusion / flow matching 解码
4. **评测集**：[[AudioCaps]] / [[Clotho]] / [[AudioSet]] subset
5. **常用指标**：FAD、IS、CLAP score、KAD
6. **与 TTS 关系**：共用 codec / VAE / 扩散后端，但条件模态不同；TTS 需 phoneme/text alignment，TTA 不需要

## 代表工作

- [[AudioLDM]]：Surrey 2023 latent diffusion 开山
- [[Tango]]：Microsoft 2023 audio latent + LLM 条件
- [[Stable Audio]]：Stability AI 商业级 long-form
- [[WavFlow]]：Meta 2026，把生成搬到波形空间
- [[Target-KL-VAE]]：把更好的 VAE 喂给 TTA 扩散

## 评测/常见数字

- AudioCaps FAD（越小越好）：AudioLDM-L ~1.7，Stable Audio Open ~1.3

## 相关概念

- [[AudioLDM]]
- [[Stable Audio]]
- [[Audio VAE]]
- [[CLAP]]
- [[TTS]]
