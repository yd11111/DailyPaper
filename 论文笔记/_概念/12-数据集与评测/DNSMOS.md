---
type: concept
aliases: [Deep Noise Suppression MOS]
---

# DNSMOS

## 定义

**Deep Noise Suppression Mean Opinion Score**：微软提出的**无参考**语音质量自动评测模型，输入一段语音波形，输出 1-5 的 MOS 分数，专门用于评估**噪声 / 失真 / 总体质量**，被广泛用于语音增强、TTS 数据筛选、合成质量自动评测。

## 核心要点

1. **无参考 (no-reference)**: 不需要 ground truth 干净语音，比 [[PESQ]] / [[STOI]] 更适用于实际场景
2. **三维输出**: SIG（信号质量）/ BAK（背景噪声）/ OVRL（总体），论文里"DNSMOS"通常指 OVRL
3. **训练数据**: ITU-T P.808 众包主观打分 → 训 CNN 回归模型
4. **典型用法**:
   - 语音增强：评估去噪效果
   - TTS 数据清洗：[[Raon-OpenTTS]] 用 DNSMOS < 2.24 作为 15% 分位过滤阈值
   - TTS 合成评测：自动评测无主观人评时的质量代理

## 常见数字

| 场景 | OVRL 典型分 |
|---|---|
| 清洁 studio 录音 (LibriHeavy) | 3.2+ |
| 普通 ASR 训练数据 (GigaSpeech) | 2.7-2.9 |
| YouTube 抓取 (Raon-YouTube-Commons) | 2.74 |
| 经过 [[Raon-OpenTTS]] 过滤的下限 | 2.24 |

## 代表工作

- DNSMOS 原论文 (Reddy et al. 2021/2022)
- [[Raon-OpenTTS]]: 用作 TTS 数据过滤的核心指标

## 相关概念

- [[UTMOS]]: 另一个无参考 MOS 预测器，更针对 TTS 自然度
- [[MOS]]: 主观 MOS 评测协议
- [[Speech Ratio]]: Raon 数据过滤的另一指标
