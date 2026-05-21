---
type: concept
aliases: [音源分离, 人声分离]
---

# Source Separation

## 定义

**Source Separation / 音源分离**：把混合音频拆分成多个独立音源（人声 / 乐器 / 背景音乐 / 多说话人）的任务。在 TTS 数据预处理中常用于**把 in-the-wild 录音的人声从背景音乐中剥离出来**。

## 核心要点

1. **典型场景**: 音乐分离（vocals / drums / bass / other）、人声 vs 背景音乐、多说话人分离
2. **常用开源工具**:
   - **UVR-MDX / UVR-MDXNet**: Ultimate Vocal Remover 系列，[[Raon-OpenTTS]] 数据管线用它做 YouTube 人声分离
   - **Demucs**: Meta 出品，强项是音乐源分离
   - **Spleeter**: Deezer 开源
3. **在 TTS 数据管线中的位置**: 通常排在 [[VAD]] 之前 — 先把人声剥出来再做 VAD / 转写
4. **挑战**: 分离后的人声可能有 artifacts，会影响 TTS 学到的音质上限

## 代表工作

- UVR-MDX
- Demucs (Défossez et al. 2021)
- [[Raon-OpenTTS]]: 在 YouTube 数据预处理中使用

## 相关概念

- [[VAD]]: 通常的后置步骤
- [[Speaker Diarization]]: 同为预处理步骤
- [[Raon-YouTube-Commons]]: 管线使用者
