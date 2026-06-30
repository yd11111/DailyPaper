---
type: concept
aliases: [AMI Corpus, AMI Meeting Corpus]
---

# AMI

## 定义
AMI (Augmented Multi-party Interaction) Meeting Corpus，多方会议录音数据集，约 100 小时，包含真实和场景模拟的英语会议录音，提供转录、说话人标注、对话行为等标注。

## 核心要点
1. 多方会议场景（通常 4 人），适合研究 turn-taking、speaker diarization
2. 含多通道麦克风阵列录音 + 近场头戴麦克风
3. 常用于对话系统预训练、会议摘要、语音分离等任务

## 代表工作
- Carletta et al. 2005: AMI 数据集论文
- [[ModeratorLM]]: 用 AMI 做对话预训练 (Stage 2)

## 相关概念
- [[Turn-taking]]
- [[Fisher]]
