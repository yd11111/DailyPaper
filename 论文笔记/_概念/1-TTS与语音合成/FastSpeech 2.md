---
type: concept
aliases: [FastSpeech2, FS2]
---

# FastSpeech 2

## 定义

微软提出的经典非自回归（NAR）TTS 模型，使用 Duration Predictor 直接预测每个 phoneme 的持续帧数，通过 Length Regulator 扩展后并行生成 Mel-Spectrogram，再接 vocoder 合成波形。

## 核心要点

1. NAR 架构，推理速度远快于 Tacotron 2 等 AR 模型
2. 依赖 Duration Predictor + forced alignment 提供时长标签
3. 额外引入 pitch / energy predictor 增强韵律控制
4. 是 NAR TTS 的代表方法，后续 NaturalSpeech / VITS 等在此基础上改进

## 代表工作

- [[FastSpeech]] (Ren et al., 2019): 初版，用 teacher-student 提取 duration
- [[FastSpeech2]] (Ren et al., 2021): 去掉 teacher，直接用 forced alignment 标签

## 评测/常见数字

- LJSpeech MOS ~3.8-4.0
- 推理 RTF << 1（非自回归并行生成）

## 相关概念

- [[Duration Predictor]]
- [[Forced Alignment]]
- [[Mel-Spectrogram]]
- [[HiFi-GAN]]
- [[Phoneme]]
