---
type: concept
aliases: [TAIL, Time-Aligned Interleaving, 时间对齐交错]
---

# TAIL (Time-Aligned Interleaving)

## 定义

[[MiniCPM-o 4.5]] 提出的 chunk-wise streaming 文-语交错策略：根据 **累积语音播放进度** 自适应控制每个 chunk 生成的文本量，使 spoken response 与 evolving environment 紧贴时间轴，避免 stale audio。

## 核心问题

- 文本生成时间 ≠ 语音播放时间，且每个 token 的 vocalization 时长是 context-dependent 的。
- 现有流式 TTS 两种方案均不理想：
  - **Large text lead** (如 [[Mini-Omni]])：先生成长文本再合成 → 文本远跑赢播放
  - **Fixed text-speech ratio** (如 [[CosyVoice|CosyVoice2]])：固定比例交错 → 假设 token vocalization 时长固定
- 全双工场景下，模型可能持续说出已 stale 的内容，不再贴合当前观测。

## 数学形式

对 [[Omni-Flow]] 的第 $k$ 个 chunk（时长 $t$），训练监督来自全双工数据中文本 token 的起止时间标注：

$$
\text{Tokens assigned to chunk } k = \{\, w_i : t_i^{\text{start}} \in [(k-1)t,\, kt) \,\}
$$

推理时模型根据累积播放时间动态决定本 chunk 文本量，使新生成内容播完后语音流接近 $kt$。

## 核心要点

1. **history-dependent 交错**: 每 chunk 的文本数量按累积 playback delay 调整，前面落后则本 chunk 少生成。
2. **Bounded look-ahead**: chunk $k$ 末尾若干文本 token 的 speech 部分推迟到 chunk $k+1$ 发音，覆盖如 `the apple` vs `the car` 的连读上下文，避免 text 显著领先 audio。
3. **训练数据要精确 token-level 起止时间**，构造成本较高（依赖强制对齐）。
4. **推理 tradeoff**: TAIL 模式 EN WER 略升至 3.93（vs Fixed 2.38），但换得 evolving environment 时延对齐。

## 代表工作

- [[MiniCPM-o 4.5]]: 提出并验证

## 评测/常见数字

| 模式 | ZH CER↓ | EN WER↓ | 适用场景 |
|---|---|---|---|
| No interleave | 1.44 | 2.70 | 离线 |
| Fixed text | 0.86 | 2.38 | turn-based 高质量 |
| Dynamic ([[TAIL]]) | 1.04 | 3.93 | full-duplex 流式 |

## 相关概念

- [[Omni-Flow]]: TAIL 是其内部子机制
- [[Forced Alignment]]: 提供训练所需的 token 起止时间标注
- [[Streaming TTS]]
