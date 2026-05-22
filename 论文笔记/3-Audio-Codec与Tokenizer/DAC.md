---
title: "High-Fidelity Audio Compression with Improved RVQGAN"
method_name: "DAC"
authors: [Rithesh Kumar, Prem Seetharaman, Alejandro Luebs, Ishaan Kumar, Kundan Kumar]
year: 2023
venue: arXiv
arxiv_id: "2306.06546"
pdf_path: "assets/papers/DAC.pdf"
library_source: "高德文献库"
source_topic: "speech-codec"
tags: [classic, codec]
created: 2026-05-19
---

# DAC: Descript Audio Codec (Improved RVQGAN)

## 📌 一句话

通用音频神经编解码器，把 44.1 kHz 音频压缩 **90×** 到 **8 kbps** 离散 token，**所有域**（语音、音乐、环境音）用同一个模型。与 [[EnCodec]] / [[SoundStream]] 并称现代 Codec 三件套。

## 🛠 核心方法

**输入 → 输出**: 44.1 kHz 原始波形 → 8 kbps 离散 token 序列（多层 RVQ 码本）

**架构组件**（按数据流顺序）:
1. **Encoder**: 卷积下采样到低帧率 latent 序列
2. **[[RVQ]] (Residual Vector Quantization)**: 多层码本逐层量化残差；引入图像域 quantization 的改进（如 FSQ 思路）
3. **Decoder**: 卷积上采样把 token 重建为波形
4. **训练目标**: 改进的对抗损失 + 多尺度重建损失（mel + STFT + waveform）+ 码本承诺损失

**关键创新**: 跨域**单一通用模型**（speech / music / env 同一权重）+ 改进 RVQ 设计让 90× 压缩仍保持高重建质量；开源 code + checkpoint 让它成为 2024+ Speech LLM 的事实离散 token 来源之一。

## 🖼 架构图

> ⚠️ DAC 论文以工程实现为主，**没有提供整体架构 overview 图**（caption 打分后无图命中阈值）。架构靠正文文字理解，主体即 Encoder + RVQ + Decoder 三段，无新结构。

## 📊 关键结果 / 评测

- **压缩率**: 44.1 kHz → 8 kbps（90×）
- **对比**: 摘要号称在所有竞争 codec（[[EnCodec]] / [[SoundStream]]）上「显著领先」，具体 MUSHRA / ViSQOL / SISDR 数字看正文
- **覆盖域**: speech / music / environmental sound 一个模型通吃
- **开源**: code + 训练好的 checkpoint 全开放

## 💡 借鉴意义（一句话）

做 Audio Codec / Speech LLM 离散 token 的人**必读**——当前最强开源通用 codec，几乎所有 2024+ Speech LLM 把它作为基线之一。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2306.06546
- PDF: [[assets/papers/DAC.pdf|本地 PDF]]
- 源目录: `speech-codec/DAC_RVQGAN.pdf`
