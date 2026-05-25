---
type: concept
aliases: [E2-TTS, E2TTS, Embarrassingly Easy TTS]
---

# E2 TTS

## 定义

Microsoft 提出的零样本 TTS 模型，核心思路是去掉 phoneme 和 [[Duration Predictor]]，直接将字符序列用 filler token 填充到与 Mel spectrogram 等长后拼接，用 [[Flow Matching]] 做 mask-and-infill 生成。架构为 flat U-Net Transformer。

## 核心要点

1. 首个证明"无 phoneme、无 duration、无 alignment"也能做 TTS 的工作
2. 局限：收敛慢、鲁棒性差（7% 样本 WER>50% 彻底失败），原因是字符和 Mel 的有效信息长度差距大导致语义-声学深度纠缠
3. 被 [[F5-TTS]] 通过 ConvNeXt 文本精炼解决了上述问题

## 代表工作

- [[F5-TTS]]: 直接改进 E2 TTS，用 ConvNeXt + DiT 替代 U-Net Transformer
- E2 TTS 原论文 (Eskimez et al., Microsoft, 2024)

## 评测/常见数字

- LibriSpeech-PC WER: ~2.95% (32 NFE), SIM-o: 0.69
- ELLA-V hard sentences WER: 8.58%（高删除率 4.82% 反映跳词问题）

## 相关概念

- [[F5-TTS]]
- [[Flow Matching]]
- [[Duration Predictor]]
- [[Voicebox]]
