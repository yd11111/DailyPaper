---
title: "SongGen: A Single Stage Auto-regressive Transformer for Text-to-Song Generation"
method_name: "SongGen"
authors: [Zihan Liu, Shuangrui Ding, Zhixiong Zhang, Xiaoyi Dong, Pan Zhang]
year: 2025
venue: arXiv
arxiv_id: "2502.13128"
pdf_path: "assets/papers/SongGen.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, music-generation]
created: 2026-05-22
---

# SongGen: Single-Stage Text-to-Song Generation

## 📌 一句话

上海 AI Lab 提出的**单阶段歌曲生成**模型——用单个 AR transformer 从歌词/文本描述直接生成包含人声和伴奏的完整歌曲，支持混合模式（人声+伴奏混合输出）和双轨模式（分离输出）。

## 🛠 核心方法

**输入 → 输出**: lyrics + text description + optional voice prompt → audio tokens → song

**架构组件**（按数据流顺序）:
1. **Condition Encoders**: 歌词编码 + 文本描述编码 + voice prompt 编码
2. **AR Transformer Decoder**: cross-attention 融合条件，生成 audio token
3. **Token Patterns**: Mixed Pro / Parallel / Interleaving 三种 token 排列策略
4. **Audio Codec Decoder**: token → waveform

**关键创新**: **单阶段** text-to-song——不需要传统的"歌词→旋律→伴奏→混音"多阶段 pipeline，一个 AR transformer 直接端到端生成。

## 🖼 架构图

![Figure 2: SongGen 框架——condition encoders + AR transformer + codec decoder](https://ar5iv.labs.arxiv.org/html/2502.13128/assets/Figure/framework1.png)

## 📊 关键结果 / 评测

- 支持中英文歌曲生成
- 双轨模式: 分离人声和伴奏

## 💡 借鉴意义（一句话）

做音乐生成的人关注——SongGen 的单阶段 text-to-song 简化了传统多阶段 pipeline，token pattern 设计值得参考。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2502.13128
- PDF: [[assets/papers/SongGen.pdf|本地 PDF]]
- 源目录: `TTS-LLM/SongGen.pdf`
