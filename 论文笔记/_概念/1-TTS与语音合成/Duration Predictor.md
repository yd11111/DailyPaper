---
type: concept
aliases: [时长预测器, Duration Modeling]
---

# Duration Predictor

## 定义

非自回归 TTS 中的关键模块：预测每个输入单元（phoneme / 字符 / token）需要展开成多少帧 Mel/codec token，从而完成"文本时间"到"语音时间"的映射。FastSpeech 系列的核心组件。

## 数学形式

通常在 phoneme encoder 输出上接一个回归头：
$$
\hat{d}_i = \mathrm{MLP}(\mathrm{Encoder}(p_i)),\quad d_i \in \mathbb{R}^+
$$
然后 length regulator 按 $\hat{d}_i$ 把 phoneme 表示展开：$h_{\text{frame}} = \mathrm{Repeat}(h_p, \lceil \hat{d}_i \rceil)$。

## 核心要点

1. NAR TTS 的灵魂模块：决定语音节奏/时长
2. 训练需要 forced alignment（[[MFA]] 等）提供 ground-truth duration
3. **"无 duration"**是新一代 TTS 的趋势：[[F5-TTS]]、[[VibeVoice]] 都不用 duration predictor

## 代表工作

- FastSpeech 2 (Ren et al. 2021)
- [[VibeVoice]]: 显式说明放弃 duration predictor，靠 LLM 上下文学习时长

## 相关概念

- [[F5-TTS]]
- [[CosyVoice]]
