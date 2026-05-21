---
type: concept
aliases: [说话人分离, 说话人日志]
---

# Speaker Diarization

## 定义

**Speaker Diarization / 说话人日志**：对一段多人对话音频，回答"什么时候谁在说话"的问题（who spoke when）。输出每段时间区间 + speaker ID。是会议转写、播客字幕、多人对话数据预处理的关键组件。

## 核心要点

1. **典型流程**: [[VAD]] → speaker embedding 提取 → 聚类 → 时间对齐
2. **常用开源工具**:
   - **PyAnnote 3.x**: HuggingFace 生态主流，[[Raon-OpenTTS]] 数据管线使用
   - **NVIDIA NeMo Diarizer**
   - **WhisperX**: Whisper + diarization 集成
3. **TTS 数据管线用途**: 多人 YouTube / 播客录音需要分离出单说话人段，才能用于 TTS 训练（避免一个 segment 里多人音色混淆）
4. **评测指标**: DER（Diarization Error Rate）

## 代表工作

- PyAnnote 系列 (Bredin et al.)
- [[Raon-OpenTTS]]: 数据管线第 3 步

## 相关概念

- [[VAD]]: 前置步骤
- [[Source Separation]]: 互补方向（一个分时间、一个分声源）
- [[Raon-YouTube-Commons]]: 管线使用者
