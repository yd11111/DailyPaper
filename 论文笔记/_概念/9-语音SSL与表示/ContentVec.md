---
type: concept
aliases: [ContentVec, Content Vector]
---

# ContentVec

## 定义
基于 HuBERT 的自监督语音表示模型，通过说话人信息解耦训练，使输出特征更纯粹地表示语言内容而非说话人身份。输出 768 维连续特征。

## 核心要点
1. 在 HuBERT 基础上通过对抗训练去除说话人信息
2. 768 维特征可直接作为语音内容表示
3. 广泛用于语音转换（SVC）和 TTS 系统的内容编码器

## 代表工作
- [[GPT-SoVITS]]: 使用 ContentVec 提取 SSL 特征作为语义 token 来源
- [[So-VITS-SVC]]: 经典 SVC 系统的内容编码器

## 相关概念
- [[HuBERT]]
- [[SSL Speech Representation]]
- [[Semantic Token]]
