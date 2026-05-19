---
type: concept
aliases: [UTMOS, UTMOSv2]
---

# UTMOS

## 定义
东京大学训练的自动 MOS 预测器，输入 waveform 输出预测 MOS 分数。

## 核心要点
1. 用 SSL 特征 + MLP
2. 高度自动化，可用于大规模筛选
3. 强模型 UTMOS 通常 4.0+

## 代表工作
- 现代 TTS 论文常用作 MOS 替代；[[OmniFlatten]] 未使用
- [[VibeVoice]]: tokenizer 重建在 LibriTTS test-other 达 UTMOS 3.724（接近 GT 3.483）

## 相关概念
- [[OmniFlatten]]
- [[VibeVoice]]
- [[PESQ]] · [[STOI]] · [[LibriTTS]]
