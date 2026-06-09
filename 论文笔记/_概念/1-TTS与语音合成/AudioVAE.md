---
type: concept
aliases: [Audio VAE, 音频变分自编码器]
---

# AudioVAE

## 定义
面向语音/音频 TTS 系统的变分自编码器，将原始波形编码为低帧率连续 latent 表示并解码回高采样率波形。与离散 codec（[[EnCodec]], [[SoundStream]]）不同，AudioVAE 保持连续 latent 空间，避免量化信息损失。

## 核心要点
1. 典型架构：因果卷积编码器 + 因果声码器式解码器（如 [[BigVGAN]]）
2. 训练通常包含对抗 loss + 重建 loss + KL 正则
3. 可增加 SSL 教师对齐（如 [[WavLM]]）使 latent 具备语义结构性
4. 训练后冻结，作为下游 AR/NAR 生成模型的目标空间

## 代表工作
- [[dots-tts]]: 128 维 @25 Hz，48 kHz 重建，HoliTok 式多目标训练（重建 + WavLM 对齐 + 多任务 supervision）
- [[VibeVoice]]: 连续 AR TTS 的 VAE 前端
- HoliTok: 多目标 VAE 训练的先驱方法

## 相关概念
- [[VAE]]
- [[BigVGAN]]
- [[EnCodec]]
- [[Continuous Autoregressive TTS]]
