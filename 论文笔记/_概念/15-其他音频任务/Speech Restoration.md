---
type: concept
aliases: [语音恢复, Speech Enhancement and Restoration]
---

# Speech Restoration

## 定义

从退化的语音信号中恢复干净语音的任务，处理的退化类型通常包括加性噪声、混响、带宽限制、削波、编码压缩伪影、丢包等——比传统 Speech Enhancement（主要处理加性噪声）覆盖范围更广。

## 核心要点

1. 与 [[Speech Enhancement]] 的区别：restoration 强调**多种退化的联合处理**，而非仅去噪
2. 现代方法通常在 SSL 模型的特征空间或 latent 空间操作，利用预训练语音表示的鲁棒性
3. 典型应用：TTS 训练数据清洗（如 LibriTTS-R、FLEURS-R、Emilia 均使用了语音恢复 pipeline）
4. 与 [[Source Separation]] 的关系：恢复处理单说话人退化，分离处理多说话人混合。DialogueSidon 等工作开始将两者统一

## 代表工作

- [[VoiceFixer]]: Liu et al. 2022，端到端语音恢复
- [[Miipher]]: Koizumi et al. 2023，基于 SSL 特征的语音恢复，用于 FLEURS-R
- Miipher-2: Karita et al. 2025，Miipher 改进版
- [[Sidon]]: Nakata et al. 2026，在 SSL 隐空间做恢复
- [[DialogueSidon]]: 扩展到对话场景的联合恢复+分离

## 相关概念

- [[Speech Enhancement]]
- [[Source Separation]]
- [[VAD]]
- [[w2v-BERT 2.0]]
