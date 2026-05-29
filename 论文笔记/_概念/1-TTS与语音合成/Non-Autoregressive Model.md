---
type: concept
aliases: [NAR, 非自回归模型, Non-Autoregressive]
---

# Non-Autoregressive Model

## 定义

非自回归（NAR）建模：一次（或固定少数次）并行预测整段输出，而非逐 token 自回归。在 codec-LM TTS 中常与 [[Autoregressive Model]] 配对——AR 生成粗（第一）码本，NAR 基于前序码本并行生成剩余细码本。优点是快，代价是难以建模逐帧细粒度依赖。

## 核心要点

1. 与 AR 对偶：AR 逐帧/逐组建模、causal mask；NAR 并行建模、full attention
2. 在 [[VALL-E]]/[[VALL-E 2]] 中，NAR 负责码本 2-8，逐码本预测（每码本以前序码本为条件）
3. 全 NAR TTS（如 [[Voicebox]]/SoundStorm）需 duration/帧对齐，会限制搜索空间、牺牲韵律
4. VALL-E 2 的 NAR 训练显式切分声学条件与目标段，从 prompt 全 8 码本抽取说话人信息，再每步随机抽一个码本 ID 优化以提效

## 代表工作

- [[VALL-E]] / [[VALL-E 2]]: AR+NAR 层次化 codec LM
- [[Voicebox]]: 全 NAR flow-matching TTS
- FastSpeech 2: 经典 NAR + duration predictor

## 评测/常见数字

- VALL-E 2 NAR 推理跑 7 次贪心生成码本 2-8

## 相关概念

- [[Autoregressive Model]]
- [[VALL-E 2]]
- [[RVQ]]
