---
type: concept
aliases: [Knowledge Distillation, 知识蒸馏, KD, Sequence-level KD]
---

# Knowledge Distillation

## 定义
用训练好的教师模型（teacher）的输出作为学生模型（student）的训练目标，使学生模型学到教师模型的"知识"。在 TTS 中，序列级知识蒸馏（sequence-level KD）指用教师 AR 模型生成的 mel-spectrogram 替代 ground-truth mel 作为训练对。

## 核心要点
1. 教师模型通常更大或更强（如 AR 模型），学生模型更快或更轻量（如 NAR 模型）
2. 序列级 KD 在 NAR 机器翻译中首先提出（Gu et al. 2017; Kim & Rush 2016）
3. FastSpeech 中，教师 AR Transformer TTS 生成的 mel 比 GT mel 更"平滑"，更适合 NAR 模型学习
4. FastSpeech 消融实验显示去掉 KD 导致 CMOS -0.325 的显著质量下降

## 代表工作
- [[FastSpeech]]: 将序列级 KD 引入 NAR TTS
- Hinton et al. 2015: 知识蒸馏的奠基论文
- Kim & Rush 2016: 序列级 KD

## 相关概念
- [[FastSpeech]]
- [[Non-Autoregressive]]
- [[Duration Predictor]]
