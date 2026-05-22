---
title: "Finite Scalar Quantization: VQ-VAE Made Simple"
method_name: "FSQ"
authors: [Fabian Mentzer, David Minnen, Eirikur Agustsson, Michael Tschannen]
year: 2023
venue: ICLR 2024
arxiv_id: "2309.15505"
pdf_path: "assets/papers/FSQ.pdf"
library_source: "高德文献库"
source_topic: "speech-codec"
tags: [classic, codec]
created: 2026-05-22
---

# FSQ: Finite Scalar Quantization

## 📌 一句话

用极简的**逐维标量量化**取代 VQ（向量量化）中的 codebook + commitment loss + EMA 更新等复杂机制，每个维度独立 round 到有限整数集，无需辅助损失、无 codebook collapse，在图像/视频/音频任务上与 VQ 持平或更优。

## 🛠 核心方法

**输入 → 输出**: encoder latent z ∈ ℝ^d → quantized ẑ ∈ 有限整数集 → decoder 重建

**架构组件**（按数据流顺序）:
1. **Encoder → d 维投影**: encoder 最后一层将 latent 投影到很少的维度（典型 d < 10）
2. **Per-channel bounding**: 每个维度用 tanh 等函数限制到 [-1, 1]，再 round 到 {-L, ..., 0, ..., L}
3. **隐式 codebook**: 总码本大小 = 各维 level 数的乘积（如 [8,6,5,5,5] → 6000 codes），无需显式存储
4. **Decoder**: 用量化后的整数向量直接重建，STE（straight-through estimator）传梯度

**关键创新**: 完全去掉 [[VQ]] 的 codebook lookup / commitment loss / EMA / 码本重置等工程 trick——量化退化为 `round(tanh(z) × L)`，一行代码替代整套 VQ 基础设施。

## 🖼 架构图

![Figure 1: FSQ (左) vs VQ (右) 对比——FSQ 将每个维度独立 bound+round，VQ 做最近邻查表](https://ar5iv.labs.arxiv.org/html/2309.15505/assets/x1.png)

## 📊 关键结果 / 评测

- MaskGIT (ImageNet 256×256): FSQ FID 4.17 vs VQ 4.34，持平
- UViM (depth/panoptic): FSQ 与 VQ 几乎一致
- 码本利用率: FSQ 接近 100%（VQ 常 collapse 到 ~40%）
- 优势在于**简单性**：无辅助损失超参调优

## 💡 借鉴意义（一句话）

做 Audio Codec / Speech Tokenizer 的人值得关注——FSQ 已被 SoundStream v2 / [[DAC]] 后续工作采纳，是替代 RVQ codebook 的有力候选方案。

## 🔗 链接

- arXiv: https://arxiv.org/abs/2309.15505
- PDF: [[assets/papers/FSQ.pdf|本地 PDF]]
- 源目录: `speech-codec/FSQ.pdf`
