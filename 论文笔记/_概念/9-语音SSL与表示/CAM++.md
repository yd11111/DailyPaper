---
type: concept
aliases: [CAM++, 3D-Speaker]
---

# CAM++

## 定义
基于密集连接时延神经网络（D-TDNN）的说话人验证模型，来自阿里达摩院 3D-Speaker 项目。用于提取固定维度的说话人 embedding 向量。

## 核心要点
1. 使用 multi-granularity pooling 聚合不同时间粒度的帧级特征
2. 在 VoxCeleb / CN-Celeb 等主流说话人验证 benchmark 上达到 SOTA
3. 广泛用于 TTS 系统中作为 speaker embedding 提取器（如 CosyVoice 2 的 flow matching 阶段）

## 代表工作
- [[CosyVoice2]]: 用 CAM++ 提取 speaker embedding 作为 flow matching 的条件输入

## 相关概念
- [[WavLM]]
- [[ECAPA-TDNN]]
