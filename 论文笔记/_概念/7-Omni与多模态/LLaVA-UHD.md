---
type: concept
aliases: [LLaVA-UHD]
---

# LLaVA-UHD

## 定义

清华 NLP 组提出的多模态视觉编码器策略，**支持任意宽高比 + 高分辨率图像** 的切片处理方案，被 [[MiniCPM-V]] / [[MiniCPM-o]] 系列采用。

## 核心要点

1. **任意宽高比**: 不强制 padding 到方形，按内容动态切片
2. **高分辨率**: 通过 slice 化 + token 压缩做到任意分辨率（如 [[MiniCPM-o 4.5]] full-duplex 模式 448×448、否则最高 2240×2240）
3. **配合 [[Resampler]]**: 每 slice 由 [[SigLIP ViT]] 编码为 1024 token，再由 Resampler 压到 64 token，得到 16× 压缩
4. **优势**: 比固定方形 + token 4× 压缩更高效

## 代表工作

- LLaVA-UHD (Guo et al. 2024, ECCV)
- [[MiniCPM-V]] 系列
- [[MiniCPM-o]] 系列

## 相关概念

- [[SigLIP ViT]]
- [[Resampler]]
- [[Vision-Language Model]]
