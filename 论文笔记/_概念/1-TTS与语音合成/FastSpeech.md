---
type: concept
aliases: [FastSpeech, FS]
---

# FastSpeech

## 定义
首个完全非自回归的并行 Mel-Spectrogram 生成 TTS 模型，通过 Feed-Forward Transformer + Duration Predictor + Length Regulator 实现快速、鲁棒、可控的语音合成。

## 核心要点
1. 用 Duration Predictor 提供硬对齐，彻底消除 AR 模型的跳词/重复问题
2. 并行生成 mel-spectrogram，mel 生成速度提升 270x，端到端 38x
3. Length Regulator 通过 alpha 参数实现平滑语速控制（0.5x-1.5x）
4. 需要先训练 AR 教师模型提取时长 + 序列级知识蒸馏

## 代表工作
- [[FastSpeech]]: Ren et al. NeurIPS 2019 -- 开创 NAR TTS 范式
- [[FastSpeech 2]]: Ren et al. ICLR 2021 -- 去除教师模型依赖，增加 pitch/energy 预测

## 评测/常见数字
- MOS 3.84 (LJSpeech, WaveGlow vocoder)，接近 Tacotron 2 的 3.86
- 50 个 hard case 0% 错误率（Tacotron 2: 24%, Transformer TTS: 34%）
- 参数量 30.1M

## 相关概念
- [[Duration Predictor]]
- [[Length Regulator]]
- [[Non-Autoregressive]]
- [[Knowledge Distillation]]
- [[FastSpeech 2]]
