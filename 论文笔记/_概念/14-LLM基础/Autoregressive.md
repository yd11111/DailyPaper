---
type: concept
aliases: [AR, 自回归, 自回归生成]
---

# Autoregressive

## 定义

自回归生成范式：逐步生成序列中的每个 token，每一步以前面所有 token 为条件预测下一个。在 TTS 中，AR 模型逐 token 生成离散音频 token（如 semantic token / codec token）。

## 数学形式

$$
p(\mathbf{y}) = \prod_{t=1}^{T} p(y_t | y_{<t}, \mathbf{x})
$$

- $\mathbf{x}$: 条件输入（文本、prompt 等）
- $y_t$: 第 $t$ 步生成的 token
- $T$: 序列总长度

## 核心要点

1. 天然支持随机采样，生成多样性和自然度好
2. 生成速度与序列长度成正比，长序列推理慢
3. 时长不易精确控制（token 数由模型自由决定），需特殊设计（如 IndexTTS2 的共享嵌入表）

## 代表工作

- [[VALL-E]]: 把 TTS 建模为条件语言模型，AR 生成 codec token
- [[IndexTTS2]]: AR + 精确时长控制 + 情感解耦
- [[CosyVoice]]: AR 生成 supervised semantic token

## 相关概念

- [[Semantic Token]]
- [[Duration Predictor]]
- [[Flow Matching]]
