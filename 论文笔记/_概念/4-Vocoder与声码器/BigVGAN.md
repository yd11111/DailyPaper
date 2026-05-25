---
type: concept
aliases: [BigVGAN-v2]
---

# BigVGAN

## 定义
基于 HiFi-GAN 架构的大规模通用神经声码器，通过增大模型容量、引入 anti-aliased multi-periodicity composition (AMP) 模块和 snake activation 实现高保真、跨域泛化的 Mel→Waveform 重建。

## 核心要点
1. 在 HiFi-GAN 基础上大幅增加参数量（112M），使用 snake activation 替代 LeakyReLU 以更好建模周期信号
2. 支持 24 kHz / 44.1 kHz 采样率，在 out-of-domain 场景（不同说话人、噪声、音乐）仍保持高质量
3. BigVGAN-v2 进一步将参数量扩展至 112M，支持 44.1 kHz，被广泛用作 TTS/Codec 系统的最终波形重建模块
4. 处理的是 Mel-Spectrogram → Waveform 的映射

## 代表工作
- [[IndexTTS2]]: 使用 BigVGANv2 作为级联系统的最终声码器
- [[Qwen3-TTS]]: 25Hz tokenizer 的流式波形重建声码器
- [[CosyVoice 2]]: 作为 flow matching 后的声码器
- [[F5-TTS]]: mel 谱图→波形重建

## 评测/常见数字
- 典型 RTF < 0.01（GPU），远低于实时线
- 在 LibriTTS 上 UTMOS > 4.0，PESQ > 3.5

## 相关概念
- [[HiFi-GAN]]
- [[Flow Matching]]
- [[DiT]]
