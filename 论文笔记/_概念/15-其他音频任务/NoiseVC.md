---
type: concept
aliases: [NoiseVC, Noise Voice Conversion]
---

# NoiseVC

## 定义
基于噪声注入的零样本语音转换方法，通过向语音表示注入噪声来擦除说话人信息，再用目标说话人 embedding 重建。

## 核心要点
1. 利用噪声扰动实现说话人-内容解耦
2. Wang & Borth, arXiv 2021

## 评测/常见数字
- VCTK ZS-VC: MOS ~3.38, Sim-MOS ~3.05（被 YourTTS 超越）

## 代表工作
- [[YourTTS]]: 作为 ZS-VC baseline 被对比

## 相关概念
- [[AutoVC]]
- [[Speaker Encoder]]
