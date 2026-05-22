---
title: "SoundStorm: Efficient Parallel Audio Generation"
method_name: "SoundStorm"
authors: [Zalán Borsos, Matt Sharifi, Damien Vincent, Eugene Kharitonov, Neil Zeghidour, Marco Tagliasacchi]
year: 2023
venue: arXiv
arxiv_id: "2305.09636"
pdf_path: "assets/papers/SoundStorm.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, speech-llm]
created: 2026-05-22
---

# SoundStorm: Efficient Parallel Audio Generation

## 📌 一句话

Google 提出的**非自回归音频生成**模型——接收 [[AudioLM]] 的 semantic token，用 **MaskGIT 式 iterative parallel decoding** 生成 [[SoundStream]] 的多层 acoustic token，比 AudioLM 快 100x 且保持语音一致性。

## 🛠 核心方法

**输入 → 输出**: semantic tokens → multi-layer acoustic tokens → waveform

**架构组件**（按数据流顺序）:
1. **Input**: AudioLM 产出的 semantic token 序列
2. **Bidirectional Transformer**: 全序列 attention，按 RVQ 层级从粗到细生成
3. **Iterative Parallel Decoding**: 每层 RVQ 做多轮 mask-predict（confidence-based），而非自回归逐帧
4. **SoundStream Decoder**: acoustic tokens → waveform

**关键创新**: 将 MaskGIT 的**迭代并行解码**思路引入 audio token 生成——每个 RVQ 层级只需几轮 unmask 即可完成，比 AudioLM 的逐帧 AR 快两个数量级。

## 🖼 架构图

![Figure 1: SoundStorm — bidirectional transformer + iterative mask-predict 架构](https://ar5iv.labs.arxiv.org/html/2305.09636/assets/x1.png)

## 📊 关键结果 / 评测

- 速度: 比 AudioLM acoustic stage 快 100x
- 质量: 保持语音一致性（long-form dialogue）
- 30 秒对话: 0.5 秒生成（vs AudioLM 的 ~1 分钟）

## 💡 借鉴意义（一句话）

做 Speech LLM / Audio Generation 的人关注——SoundStorm 证明了非自回归 iterative decoding 在音频生成中的巨大速度优势，[[MaskGCT]] 等后续工作继承了这一思路。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2305.09636
- PDF: [[assets/papers/SoundStorm.pdf|本地 PDF]]
- 源目录: `TTS-LLM/soundstorm.pdf`
