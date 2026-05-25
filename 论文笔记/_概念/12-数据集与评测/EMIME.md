---
type: concept
aliases: [EMIME Dataset, EMIME Bilingual Dataset]
---

# EMIME

## 定义
EMIME (Effective Multilingual Interaction in Mobile Environments) 双语语音数据集，包含同一说话人分别说中文和英文的录音，用于跨语言 TTS 和 S2ST 评测。

## 核心要点
1. 包含 14 个说话人（7 女 + 7 男）的中英双语录音
2. 每人 25 个双语句对，共 350 个测试样本
3. 因为同一说话人说两种语言，可以做精确的跨语言说话人相似度评测
4. 是跨语言 TTS/S2ST 领域常用的小规模评测集

## 评测/常见数字
- VALL-E X 在 EMIME 上的 SMOS: 4.12 (S2ST), 4.00 (XTTS)

## 代表工作
- [[VALL-E-X]]: 使用 EMIME 评测跨语言 TTS 和 S2ST

## 相关概念
- [[S2ST]]
- [[Cross-Lingual TTS]]
- [[LibriSpeech]]
