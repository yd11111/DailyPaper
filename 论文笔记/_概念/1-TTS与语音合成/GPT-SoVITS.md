---
type: concept
aliases: [GPT-SoVITS, GPTSoVITS, GPT SoVITS]
---

# GPT-SoVITS

## 定义
中文社区最流行的开源零样本/少样本 TTS 方案，结合 GPT 风格的 12 层 Transformer 自回归预测 [[Semantic Token]]（1024 码本）与 [[VITS]] 架构（含 [[MRTE]] + [[Normalizing Flow]] + [[HiFi-GAN]]）解码为波形。仅需 5 秒参考音频即可零样本克隆。

## 核心要点
1. 两阶段架构：GPT（Text2Semantic，512d / 8 头 / 12 层）→ SoVITS（Semantic2Waveform，VITS 变体）
2. GPT 使用非对称因果掩码：文本双向可见、音频因果，类似 Prefix LM
3. MRTE（Multi-Reference Timbre Encoder）通过 [[Cross-Attention]] 融合内容和音色
4. 支持 [[DPO]] 对齐训练（reference-free，beta=0.2）
5. GitHub 57.8k+ stars，中文语音克隆事实标准
6. V1→V4/V2ProPlus 多版本迭代，RTF 0.014（4090）

## 评测/常见数字
- LibriSpeech test-clean: WER 5.13%, SIM-O 0.405（CosyVoice 2 为 2.57% / 0.764）
- RTF: 0.014（4090）/ 0.028（4060Ti）/ 0.526（M4 CPU）

## 代表工作
- [[GPT-SoVITS]]: 本体（开源项目，无正式论文）
- [[CosyVoice 2]]: 学术评测中的主要对比对象

## 相关概念
- [[VITS]]
- [[VALL-E]]
- [[ContentVec]]
- [[HiFi-GAN]]
- [[BigVGAN]]
- [[RVQ]]
- [[Cross-Attention]]
- [[DPO]]
