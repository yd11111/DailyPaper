---
type: concept
aliases: [SPGI Speech]
---

# SPGISpeech

## 定义

**S&P Global**（Kensho）发布的金融领域英文 ASR 数据集，源自上市公司业绩电话会议（earnings calls），约 **5K 小时**（v1）+ **889 小时**（v2.0），人工转写质量高。Kensho User Agreement 授权。

## 核心要点

1. **领域**: 金融会议，演讲者多元（CEO/分析师），口音和专业术语丰富
2. **质量**: [[Whisper]] WER 0.03（v1）/ 0.08（v2.0），全 Pool 中最低 — 因为是**人工精校转写**
3. **License**: Kensho UA，需注册同意（非纯 CC 但可学术使用）
4. **典型用途**: 长音频 ASR / 领域适配 / TTS 数据

## 代表工作

- SPGISpeech 数据集论文（O'Neill et al. 2021）
- [[Raon-OpenTTS]]: 作为 Pool 来源之一

## 相关概念

- [[GigaSpeech]]: 同档英文 ASR
- [[Raon-OpenTTS-Pool]]
