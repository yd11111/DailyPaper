---
type: concept
aliases: [AR, AR Model, 自回归模型, Autoregressive]
---

# Autoregressive Model

## 定义
自回归模型，按时间步逐一生成 token/帧，每步以前序生成结果为条件。在 TTS 中表现为逐 token 预测语音序列。

## 数学形式

$$
p(x_{1:T}) = \prod_{t=1}^{T} p(x_t | x_{<t})
$$

## 核心要点
1. **优点**: 生成质量高，天然支持变长序列，无需时长预测
2. **缺点**: 推理慢（不可并行），可能出现 error accumulation
3. 代表性 AR TTS: Tacotron 2、VALL-E、CosyVoice

## 代表工作
- [[WaveNet]]: 首个原始波形级别自回归音频生成模型
- [[VALL-E]]: AR 语言建模做 TTS
- [[CosyVoice]]: AR LLM + Flow Matching

## 相关概念
- [[Duration Predictor]]
- [[Error Accumulation]]
