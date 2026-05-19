---
type: concept
aliases: [Omni-Flow, 全双工 Omni 流式框架]
---

# Omni-Flow

## 定义

[[MiniCPM-o 4.5]] 提出的统一流式建模框架：用 **共享毫秒级时间轴** 把视/听/说三路 stream 编排成单一因果 token 序列，使 perception 与 response 在 token 级并行，实现真正的 [[Full-Duplex|全双工]] 多模态交互。基于 [[时分多路复用]] (TDM) 思想。

## 数学形式

把连续交互划分为时长 $t$ 的 chunk。第 $k$ 个 chunk 内：
- env-visual → 视觉 token $\mathbf{v}^{k}$
- env-audio → 音频 token $\mathbf{a}^{k}$
- out-stream → 输出 token $\mathbf{o}^{k}$（无输出时仅含 `[listen]` 特殊 token）

$$
\mathbf{g}_{k} = [\mathbf{v}^{k};\, \mathbf{a}^{k};\, \mathbf{o}^{k}]
$$

完整交互序列即 $\mathbf{g}_{1}, \mathbf{g}_{2}, \dots, \mathbf{g}_{K}$ 拼接，喂给标准 causal LM。

## 核心要点

1. **三路 time-aligned stream**: env-visual / env-audio / out-stream。User request 不再是 privileged role，统一作为 always-on 环境的一部分。
2. **chunk 内顺序**: 先消化新感知 token，再生成输出 token → 每次输出条件于最新观测。
3. **`[listen]` 特殊 token**: 模型自主决定是否输出，自然支持 [[Proactive Behavior]] 与 turn-taking，不需要外置 [[VAD]]。
4. **三个设计维度（消融过）**:
   - Temporal granularity: chunk size 1.0 s 最优（latency-capacity tradeoff）
   - Boundary: explicit special token > implicit
   - Control: Listen-Speak (LS, 先 binary 控制再生成内容) > Listen-Text (LT, 共享空间)
5. **统一兼容**: 同一架构同时支持 turn-based 和 full-duplex 模式，可自由切换。

## 代表工作

- [[MiniCPM-o 4.5]]: 首次提出并实现，9B 参数 + 12 GB RAM 端侧实时全双工。

## 相关概念

- [[TAIL|Time-Aligned Interleaving]]: Omni-Flow 内部的文-语对齐子机制
- [[Proactive Behavior]]: Omni-Flow 自然支持的行为类型
- [[时分多路复用]]: 借鉴的工程思想
- [[Full-Duplex]]
- [[VAD]]
