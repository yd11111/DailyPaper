---
type: concept
aliases: [Comparative MOS]
---

# CMOS

## 定义

**Comparative Mean Opinion Score**：TTS 主观评测的 A/B 偏好打分协议。听众两两对比 A 和 B 两段合成语音，按 7 点制（−3 到 +3）打出偏好程度，**正数代表更偏好 A**。常用于评估**自然度（naturalness）**和**整体质量**。

## 核心要点

1. **相对而非绝对**：避免不同评测员对绝对 MOS 校准不同的问题
2. **7 点尺度**: −3=much worse, −2=worse, −1=slightly worse, 0=about the same, +1=slightly better, +2=better, +3=much better
3. **显著性**: ±0.05 通常已被视为有意义差异，±0.5 是"明显更好"
4. **典型样本数**: 每条件 30 个 item × 6 个 annotator（[[Raon-OpenTTS]] 用 Amazon MTurk）
5. **锚点选择**: 通常以 baseline 或 ground truth 为锚点（0），其余模型为相对值；锚点选择对结果偏向有显著影响

## 代表工作

- [[Raon-OpenTTS]]: 用 Raon-1B 为锚点对比 [[F5-TTS]]、[[CosyVoice 3]]、[[Qwen3-TTS]] 等
- 几乎所有近代 TTS 论文（[[VALL-E]] / [[NaturalSpeech 2]] / [[F5-TTS]]）都报告 CMOS

## 相关概念

- [[MOS]]: 绝对评分
- [[SMOS]]: 主观说话人相似度评分
- [[UTMOS]]: 自动 MOS 预测，常作为 CMOS 的客观代理
