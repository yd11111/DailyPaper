---
type: concept
aliases: [VoxCPM-Emilia]
---

# VoxCPM

## 定义

面壁智能 (ModelBest) / 清华 THUHCSI 联合开发的 0.5B tokenizer-free [[Continuous Autoregressive TTS|C-AR]] TTS 系统。通过可微分 [[FSQ]] 瓶颈实现语义-韵律规划与声学渲染的结构性分离，无需外部 speech tokenizer，端到端以 [[Flow Matching]] 目标训练。1.8M 小时双语数据训练；95K 小时版本叫 VoxCPM-Emilia。

## 核心要点

1. **层次化架构**: TSLM (基于 [[MiniCPM]]-4 初始化) → [[FSQ]] 半离散瓶颈 → RALM 残差补回声学细节 → LocDiT 局部扩散解码
2. **FSQ 作为正则化**: 不同于传统 [[RVQ]] 作为预测目标，FSQ 仅作为中间可微归纳偏置约束隐状态空间
3. **残差设计**: RALM 专门补回 FSQ 量化丢失的细粒度声学信息（音色、环境等），与 TSLM 形成自然分工
4. **上下文感知**: 从纯文本推断合适的语气风格，支持方言克隆、录音条件克隆

## 评测/常见数字

| Benchmark | EN WER | EN SIM | ZH CER | ZH SIM |
|-----------|--------|--------|--------|--------|
| SEED-TTS-EVAL | 1.85% | 72.9 | 0.93% | 77.2 |
| CV3-EVAL | 4.04% | - | 3.40% | - |

- RTF 0.17 on RTX 4090
- EN S-MOS 4.18, ZH N-MOS 4.10（开源系统最优之一）

## 代表工作

- [[VoxCPM]]: 本体论文 (arXiv:2509.24650)
- [[SemaVoice]]: 后续相关工作

## 相关概念

[[Continuous Autoregressive TTS]] / [[FSQ]] / [[Flow Matching]] / [[DiTAR]] / [[Emilia]] / [[MiniCPM]]
