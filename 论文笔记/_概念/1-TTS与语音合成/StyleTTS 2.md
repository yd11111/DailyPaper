---
type: concept
aliases: [StyleTTS2]
---

# StyleTTS 2

## 定义

基于风格扩散和对抗训练的端到端 TTS 系统，通过大规模 SLM（speech language model）作为判别器实现高质量的零样本语音合成和风格控制。

## 核心要点
1. 使用 diffusion model 对 style vector 建模
2. 引入大规模 SLM 作为对抗训练的判别器
3. 支持少数据集上的高质量合成
4. 在 LJSpeech / LibriTTS / VCTK 上训练

## 评测/常见数字
- LibriSpeech test-clean: WER 2.49, Sim-O 0.38, CMOS -0.21（相对 NaturalSpeech 3）

## 代表工作
- 本身即是代表工作

## 相关概念
- [[VITS]]
- [[Classifier-Free Guidance]]
- [[Zero-shot TTS]]
