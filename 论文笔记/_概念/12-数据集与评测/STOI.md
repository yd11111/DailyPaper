---
type: concept
aliases: [Short-Time Objective Intelligibility]
---

# STOI

## 定义

时频加权噪声语音的短时客观可懂度评测（Taal et al. 2010）：取值 0–1，越高越可懂。常用于语音增强 / 分离 / codec 评测。

## 核心要点

1. 关注**时间细节 / 可懂度**，离散 RVQ codec 在此项有优势
2. 连续 latent / 低帧率 codec 在 STOI 上常吃亏
3. [[VibeVoice]] 在 STOI 上略输离散 [[EnCodec]]（0.828 vs 0.939），但 PESQ/UTMOS 反超

## 代表工作

- 原 paper (Taal et al. ICASSP 2010)
- [[VibeVoice]] tokenizer 重建评测

## 评测/常见数字

- 高质量重建: 0.85–0.95
- VibeVoice tokenizer (test-clean): 0.828

## 相关概念

- [[PESQ]]
- [[UTMOS]]
