---
type: concept
aliases: [CosyVoice 3 Eval, CV3-Eval Benchmark]
---

# CV3-Eval

## 定义

CosyVoice 3 提出的多语言零样本语音合成评测基准，覆盖 9 种语言，包含多语言/跨语言/情感/方言/困难样本等多维度测试。

## 核心要点

1. **多语言语音克隆**: 9 语言（zh/en/ja/ko/de/fr/ru/it/es）× 500 样本，来源于 Common Voice + FLEURS，含噪声和长静音
2. **困难样本**: 中英文各一套 hard test set（罕见词、绕口令、领域术语）
3. **跨语言语音克隆**: 4 语言（zh/en/ja/ko）两两交叉
4. **情感克隆**: happy/sad/angry，分为"文本相关"和"文本无关"两个子集
5. **方言评测**: 18 种中国方言/口音，仅主观评测（无可靠客观方言评测方法）
6. **主观评测**: 表达性克隆 + 语音续写（120 样本，前 3 秒作 prompt）

## 代表工作

- [[CosyVoice3]]: 提出该 benchmark

## 相关概念

- [[Seed-TTS-eval]]
- [[Common Voice]]
- [[Fleurs]]
- [[MOS]]
- [[WER]]
