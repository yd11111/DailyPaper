---
type: concept
aliases: [VoxCeleb1, VoxCeleb2, VoxCeleb]
---

# VoxCeleb

## 定义
大规模说话人识别/验证数据集，从 YouTube 名人访谈中提取。VoxCeleb1 含 1,251 说话人，VoxCeleb2 含 6,112 说话人。广泛用于训练 speaker encoder / speaker verification 模型。

## 核心要点
1. VoxCeleb1 (Nagrani et al., 2017): 1,251 说话人, ~352h
2. VoxCeleb2 (Chung et al., 2018): 6,112 说话人, ~2,442h
3. 录制条件多样（噪声、混响、不同设备），适合训练鲁棒的 speaker encoder
4. 是 speaker verification/embedding 的标准训练集

## 代表工作
- [[YourTTS]]: Speaker Encoder (H/ASP) 在 VoxCeleb2 上训练
- [[Speaker Encoder]]: 大多数 speaker encoder 以 VoxCeleb 为训练数据

## 相关概念
- [[Speaker Encoder]]
- [[VCTK]]
