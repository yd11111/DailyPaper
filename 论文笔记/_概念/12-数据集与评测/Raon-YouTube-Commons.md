---
type: concept
aliases: [Raon YT-C, Raon YouTube Commons]
---

# Raon-YouTube-Commons

## 定义

[[Raon-OpenTTS]] 团队基于公开的 **YouTube-Commons**（CC BY 4.0 创作共用 YouTube 视频集合）做端到端预处理后得到的 **335K 小时、141.7M 段**英文语音子集，是 [[Raon-OpenTTS-Pool]] 中占比最大的来源（>50%）。

## 预处理管线

1. **音频标准化**: 16 kHz mono + loudness normalization
2. **[[Source Separation]] / 人声分离**: UVR-MDX
3. **[[Speaker Diarization]]**: PyAnnote 3.1
4. **[[VAD]]**: Silero VAD，输出 3–30 秒切片
5. **转写**: [[Whisper]]-large-v3
6. **存储**: 64 kbps Opus

## 质量统计

| 指标 | 值 |
|---|---|
| [[DNSMOS]] | 2.74 |
| [[Whisper]] WER | 0.30 |
| [[Speech Ratio]] | 0.90 |
| 过滤后保留率 | 82.9% |

## 核心要点

1. **License 友好**: CC BY 4.0，可商用
2. **效果**: [[Raon-OpenTTS]] Table 10 显示加入 YC 后 Clean / Wild / Expressive 场景全面提升，但 Noisy WER 反伤（4.21→6.79）
3. **意义**: 第一个端到端开源 + 公开 YouTube 量级 TTS 训练子集

## 代表工作

- [[Raon-OpenTTS]]: 发布与使用

## 相关概念

- [[Raon-OpenTTS-Pool]]: 父数据池
- [[Raon-OpenTTS-Core]]: 过滤后用于训练的子集
