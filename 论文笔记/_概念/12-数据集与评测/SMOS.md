---
type: concept
aliases: [Similarity MOS]
---

# SMOS

## 定义

**Similarity Mean Opinion Score**：TTS 主观评测的**说话人相似度**评分协议。听众听一段参考语音 + 一段合成语音，按 5 点制（Bad / Poor / Fair / Good / Excellent）打分判断"说话人是否一致"。

## 核心要点

1. **专门评估 [[Zero-shot TTS]] 音色克隆质量**
2. **5 点制（1-5）**: 1=Bad, 2=Poor, 3=Fair, 4=Good, 5=Excellent
3. **典型分数**:
   - 强模型 3.5+
   - [[Raon-OpenTTS]]-1B 在 Raon-Eval Overall 拿 3.70
   - Ground truth re-recording 通常 4.0+
4. **客观对应物**: SIM-O / SIM-R（speaker embedding cosine）
5. **典型样本数**: 同 [[CMOS]]，30 item × 6 annotator

## 代表工作

- [[Raon-OpenTTS]]: 主表 Table 8 报告 SMOS
- [[VALL-E]] / [[CosyVoice 2]] / [[F5-TTS]] 等几乎所有 zero-shot TTS

## 相关概念

- [[MOS]] / [[CMOS]]: 自然度评测
- [[Zero-shot TTS]]: 主要应用场景
