---
type: concept
aliases: [MUSDB, MUSDB18-HQ]
---

# MUSDB18

## 定义

由 Rafii et al. (2017) 发布的音乐源分离 benchmark 数据集："MUSDB18 - a corpus for music separation"。包含 150 首多乐器音乐曲目（100 train / 50 test），每首提供 mixture + 4 个 stems（vocals / drums / bass / other）的对齐版本。是 music source separation 任务的标准评测集。在 [[LoSATok]] 中被用作音乐域 audio reconstruction 评测。

## 规模

- 150 曲（100 train / 50 test）
- 总时长约 10 小时
- 44.1 kHz stereo + stems

## 核心要点

1. **音乐源分离金标准**：从 Demucs / Open-Unmix / HT-Demucs 到现代基于扩散的分离模型都报 MUSDB18 上的 SDR。
2. **作为重建评测集**：在 audio codec / tokenizer 工作中被借用作"音乐重建质量"评测（如 [[LoSATok]] / [[DashengTokenizer]] 在 MUSDB 上算 Mel / STFT distance）。
3. **HQ 版本**：MUSDB18-HQ 提供未压缩 wav，质量更好。

## 代表工作

- Rafii et al. (2017)
- 被几乎所有 music source separation 论文使用。
- 在 [[LoSATok]] / [[DashengTokenizer]] 中作 music 重建评测。

## 相关概念

- [[Source Separation]]
- [[SDR]] (Signal-to-Distortion Ratio)
- [[AudioSet]] / [[MTG-Jamendo]]：其他常用音乐数据集。
