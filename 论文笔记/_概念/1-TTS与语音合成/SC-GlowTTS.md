---
type: concept
aliases: [SC-GlowTTS, Speaker Conditional Glow-TTS]
---

# SC-GlowTTS

## 定义
首个基于 Flow 的零样本多说话人 TTS 模型，在 Glow-TTS 基础上引入说话人条件化，实现零样本 voice cloning。YourTTS 同一作者（Casanova et al.）的前序工作。

## 核心要点
1. 基于 Glow-TTS 的 normalizing flow 架构
2. 展示仅用 11 个 VCTK 说话人也能实现一定零样本能力
3. Interspeech 2021

## 评测/常见数字
- VCTK ZS-TTS: SECS 0.804, MOS 3.78, Sim-MOS 3.99

## 代表工作
- [[YourTTS]]: 直接后续工作，改用 VITS 架构并大幅提升

## 相关概念
- [[VITS]]
- [[Affine Coupling Layer]]
- [[Monotonic Alignment Search]]
