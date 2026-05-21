---
type: concept
aliases: [Voice Activity Detection, 语音活动检测]
---

# VAD

## 定义

**Voice Activity Detection / 语音活动检测**：判断一段音频的每一帧是「语音」还是「非语音（沉默/噪声/音乐）」的二分类任务。是几乎所有语音管线的前置组件（ASR / TTS / 全双工对话）。

## 核心要点

1. **典型输出**: 逐帧的 0/1 标签或概率
2. **常用实现**:
   - **Silero VAD**: 轻量级开源 VAD，[[Raon-OpenTTS]] 数据管线用它做 segment 切分
   - **PyAnnote VAD**: HuggingFace 生态，更准但更重
   - **WebRTC VAD**: 工业老牌
3. **关键应用**:
   - **数据预处理**: 把长音频切成 3-30 s 训练段（[[Raon-YouTube-Commons]]）
   - **数据筛选**: [[Speech Ratio]] = VAD 正帧占比，用于过滤非语音占比高的段
   - **流式 ASR/对话**: 检测 End-of-Speech (EOS)、触发响应
   - **全双工系统**: turn-taking 决策

## 代表工作

- Silero VAD（开源）
- PyAnnote (Bredin et al.)
- [[Raon-OpenTTS]]: 数据管线核心组件

## 相关概念

- [[Speech Ratio]]: VAD 的直接衍生指标
- [[Speaker Diarization]]: 通常 VAD 是其前置步骤
- [[Full-Duplex]]: 全双工依赖 VAD 做 turn-taking
