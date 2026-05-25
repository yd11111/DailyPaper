---
title: "FlexiCodec: A Dynamic Neural Audio Codec for Low Frame Rates"
method_name: "FlexiCodec"
authors: []
year: 2026
venue: ICLR 2026 submission
arxiv_id: null
pdf_path: "assets/papers/FlexiCodec.pdf"
library_source: "高德文献库"
source_topic: "TTS-LLM"
tags: [classic, codec]
created: 2026-05-22
---

# FlexiCodec: Dynamic Neural Audio Codec for Low Frame Rates

## 📌 一句话

提出 **dynamic frame rate** 的神经音频 codec——根据语音内容复杂度动态调整帧率（静音/简单段低帧率，复杂段高帧率），在极低帧率下保持高质量重建，对 Speech LLM 更友好（更短 token 序列）。

## 🛠 核心方法

**输入 → 输出**: audio waveform → dynamic-rate discrete tokens → reconstructed waveform

**架构组件**（按数据流顺序）:
1. **Encoder**: CNN 下采样
2. **Dynamic Frame Rate Module**: 根据内容复杂度自适应选择帧率
3. **VQ/RVQ**: 量化
4. **Decoder**: 重建波形

**关键创新**: 打破了固定帧率的限制——语音中大量低信息量段（如静音、稳态元音）不需要高帧率编码，动态调整可显著减少 token 数量。

## 📊 关键结果 / 评测

- 匿名投稿（ICLR 2026），具体数字见论文 Table 1-2

## 💡 借鉴意义（一句话）

做 Audio Codec / Speech LLM 的人关注——动态帧率是减少 speech token 序列长度的另一种思路（与 [[SNAC]] 的 multi-scale 互补）。

## 🔗 链接

- PDF: [[assets/papers/FlexiCodec.pdf|本地 PDF]]
- 源目录: `TTS-LLM/FLEXICODEC- A DYNAMIC NEURAL AUDIO CODEC FOR LOW FRAME RATES.pdf`
