---
type: concept
aliases: [Crisper Whisper]
---

# CrisperWhisper

## 定义

社区微调的 [[Whisper]] 变种（nyrahealth 等团队 2024 年放出），目标是生成"crisp"逐字转写——保留 filler word（uh, um）、停顿、修正等不被原 Whisper 默默删除的语音现象。常用于会议转写、医疗记录、对话研究等场景。

## 核心要点

1. **不"美化"原话**：原版 Whisper 会自动平滑掉 filler word，CrisperWhisper 把它们都保留下来
2. **基于 large-v2/v3 微调**：在带 disfluency 标注的数据上 fine-tune
3. **更精确的词级时间戳**：在严格逐字标注数据上训练，时间戳偏差更小
4. **对会议 / 对话研究友好**：可用于研究 turn-taking、停顿、重复

## 代表工作

- HuggingFace `nyrahealth/CrisperWhisper`
- [[PAREDA]]：多口音数据集评测中作为强 ASR 基线

## 评测/常见数字

- 在含 disfluency 标注的会议数据上 WER（包含 filler）比原版 Whisper 低 5–10 个百分点

## 相关概念

- [[Whisper]]
- [[Disfluency]]
