---
type: concept
aliases: [Whisper, Whisper-Small, Whisper-Large, Whisper-Large-V3, Whisper-Turbo]
---

# Whisper

## 定义
OpenAI 2022 发布的多语种通用 ASR 模型，端到端 encoder-decoder Transformer，68 万小时弱监督训练。

## 核心要点
1. Multilingual: 99 种语言
2. 强鲁棒：长音频、口音、噪声
3. Whisper-Large-V3: LibriSpeech test-clean WER 1.82
4. Distil/Turbo 版本作低延迟蒸馏

## 代表工作
- 几乎所有 TTS 论文都用 Whisper 反转后算 WER 衡量发音可懂度
- [[OmniFlatten]] ASR 评测基线
- [[VibeVoice]]: 用 Whisper-large-v3 反解长对话音频得到 WER（VibeVoice-1.5B 1.11 / 7B 1.29）

## 相关概念
- [[OmniFlatten]]
- [[VibeVoice]]
- [[Paraformer]]
- [[ASR]]
