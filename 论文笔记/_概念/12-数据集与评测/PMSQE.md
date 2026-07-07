---
type: concept
aliases: [Perceptual Metric for Speech Quality Evaluation, 感知加权频谱距离]
---

# PMSQE

## 定义
Perceptual Metric for Speech Quality Evaluation，一种感知加权的频谱距离度量，模拟人耳对不同频率和响度变化的敏感度差异。作为可微损失函数用于语音增强模型训练，比纯频谱 MSE 更符合主观听感。

## 核心要点
1. 结合了心理声学模型的频率加权和响度加权
2. 作为损失函数可微，适合端到端训练
3. 与 PESQ 的相关性高于纯频谱损失
4. 常与频谱损失、SI-SNR 等组合使用

## 代表工作
- [[LMPAN]]: 训练损失组合中包含 PMSQE loss（权重 0.8，是权重最高的组分）

## 相关概念
- [[PESQ]]
- [[SI-SNR]]
- [[STOI]]
