---
type: concept
aliases: [pyannote, pyannote-audio, pyannote.audio]
---

# pyannote

## 定义
开源的 Python 语音分析工具包（pyannote.audio），提供说话人分割（Speaker Diarization）、语音活动检测（VAD/SAD）、说话人变化检测（SCD）、重叠语音检测（OSD）等功能。基于 PyTorch，提供预训练模型。

## 核心要点
1. 支持 VAD / SAD / SCD / OSD / Speaker Diarization
2. 模型通过 HuggingFace Hub 分发（如 `pyannote/segmentation-3.0`）
3. 广泛用于 TTS 数据处理管线的音频分段和清洗

## 代表工作
- [[PilotTTS]]: 数据管线 Stage 1 使用 pyannote 做 SAD+SCD，Stage 2 使用 segmentation-3.0 做 OSD

## 相关概念
- [[VAD]]
- [[Speaker Diarization]]
- [[Source Separation]]
