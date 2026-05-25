---
type: concept
aliases: [Spear TTS]
---

# SpearTTS

## 定义
Google 提出的两阶段 TTS 系统，先用 text-to-semantic 模型生成 HuBERT 语义 token，再用 semantic-to-acoustic 模型生成声学 token。是 VALL-E 之外的另一条 LLM-based TTS 路线。

## 核心要点
1. 使用 HuBERT 语义 token 作为中间表示
2. 两阶段自回归生成：text → semantic token → acoustic token
3. CosyVoice 在相同框架下证明 S3 token 优于 HuBERT token

## 评测/常见数字
- LibriTTS test-clean WER: 6.14%, SS: 51.71（vs CosyVoice 的 3.17%, 69.49）

## 代表工作
- [[CosyVoice]]: 对比基线

## 相关概念
- [[HuBERT]]
- [[VALL-E]]
- [[Discrete Audio Token]]
