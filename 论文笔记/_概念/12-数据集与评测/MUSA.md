---
type: concept
aliases: [MUSA Benchmark, Multilingual Selective Auditory Attention]
---

# MUSA

## 定义

2026 年提出的 LALM 选择性听觉评测 benchmark（来自论文 *Can Large Audio Language Models Ignore Multilingual Distractors?*）。把目标语音与外语 distractor 在不同 [[SNR]] 下混合，看 [[LALM]] 能否忽略干扰只听目标——同时把"选择性听觉"与"语言识别能力"在评分中解耦。

## 核心要点

1. **任务定位**：cocktail-party 场景下的 LALM 鲁棒性测试
2. **多语种 distractor**：用外语干扰人声，模拟真实多语言环境
3. **多 SNR 设置**：跨 SNR 看模型性能曲线
4. **diagnostic 错误分类**：把模型答案标注为"来自目标声 / 来自干扰声 / 编造"以分析失败模式
5. **指标**：MCQ ACC + WER + SDR + ECE，多维度评测
6. **被测模型**：[[Audio-Flamingo]] 等主流 LALM

## 代表工作

- 原论文 (arXiv 2605.17225)

## 评测/常见数字

- 主流 LALM 在 0 dB SNR + 多语种干扰下，MCQ 准确率从 cleam 70%+ 降至 < 50%

## 相关概念

- [[LALM]]
- [[Audio-Flamingo]]
- [[SALMONN]]
- [[Cocktail Party Problem]]
