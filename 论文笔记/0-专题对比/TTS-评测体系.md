---
title: TTS 评测体系
type: 专题对比
domain: TTS
tags: [evaluation, metrics, benchmark, tts]
created: 2026-05-25
last_updated: 2026-05-25
---

# TTS 评测体系

## 评测维度全景

TTS 评测需要覆盖多个正交维度，单一指标无法全面衡量系统质量。

| 维度 | 测什么 | 主要指标 | 典型 benchmark |
|------|--------|---------|---------------|
| **语音质量** | 自然度、清晰度、无伪影 | MOS / UTMOS / DNSMOS | LJSpeech / LibriTTS |
| **说话人相似度** | 与参考音色的匹配度 | SIM-O / SECS / EER | Seed-TTS-eval / LibriSpeech |
| **内容准确性** | 发音正确、无漏字多字 | WER / CER (ASR 转录) | 自定义 hard case 集 |
| **韵律自然度** | 停顿、重音、语调 | MOS (韵律) / F0 RMSE | 需专家评审 |
| **多语言** | 跨语言泛化 | 各语言独立 WER/MOS | Seed-TTS-eval (zh/en) |
| **鲁棒性** | 长文本、特殊符号、重复 | 成功率 / 完成率 | EmergentTTS-Eval |
| **效率** | 推理速度、资源占用 | RTF / 首包延迟 / 显存 | 自测（硬件绑定） |

## 核心指标详解

### 主观指标

#### [[MOS]] (Mean Opinion Score)

- **定义**: 人工评分 1-5 分（1=差, 5=优秀），取均值
- **优点**: 最直接反映人耳感知
- **缺点**: 成本高（需 20+ 评审员 × 数百条）、不可复现（不同评审池结果不同）、存在评分膨胀（新系统普遍被评 4.0+）
- **变体**: CMOS（对比 MOS）、DMOS（退化 MOS）
- **使用情况**: [[CosyVoice3]]、[[Qwen3-TTS]]、[[VoxCPM]] 等所有主要系统都报告 MOS

#### [[MUSHRA]] (MUltiple Stimuli with Hidden Reference and Anchor)

- **定义**: 在 0-100 范围内同时对多个系统评分，包含隐藏的参考和锚点
- **优点**: 对比性强，评审一致性更高
- **缺点**: 设计复杂（需要合适的锚点）
- **使用情况**: [[StepAudio2.5]] 报告了 MUSHRA

### 客观指标 — 智能度

#### [[WER]] / [[CER]] (Word/Character Error Rate)

- **定义**: 用 ASR 模型转录合成语音，与原文计算编辑距离
- **依赖**: ASR 模型选择（[[Whisper]] large-v3 是目前最常用的）
- **注意**: WER 低不代表听感好，只代表"说对了字"
- **典型值**: 顶级系统在 LibriSpeech test-clean 上 WER < 3%

#### Seed-TTS-eval WER/CER

- **定义**: 字节 SeedTTS 提出的标准化评测集，包含中英文 hard case
- **优点**: 有公开评测集、困难样本（绕口令、长句、多语码切换）
- **使用情况**: [[CosyVoice2]]、[[CosyVoice3]]、[[IndexTTS2]] 广泛采用

### 客观指标 — 说话人相似度

#### [[SIM-O]] (Speaker Similarity - Objective)

- **定义**: 用说话人验证模型（如 WavLM-TDNN / ECAPA-TDNN）计算参考音频与合成音频的余弦相似度
- **范围**: 0-1，越高越好
- **注意**: 不同 speaker encoder 给出的绝对值不可比；应使用相同 encoder 做系统间对比
- **典型值**: 零样本场景 SIM-O > 0.70 算可用，> 0.80 算优秀

#### [[SECS]] (Speaker Embedding Cosine Similarity)

- 同 SIM-O 的变体命名，本质相同

### 客观指标 — 自动 MOS

#### [[UTMOS]] (Universal TTS MOS Predictor)

- **定义**: 训练的 MOS 预测模型，输入合成语音输出预测 MOS
- **优点**: 免费、可复现、快速
- **缺点**: 与真实 MOS 相关性有限（尤其在非英语和极端质量区间）
- **典型值**: 4.0+ 表示质量不错

#### [[DNSMOS]] (Deep Noise Suppression MOS)

- **定义**: 微软提出的自动 MOS，侧重噪声和信号质量
- **用途**: 评估语音增强后的质量，TTS 中较少单独使用

### 效率指标

#### [[RTF]] (Real-Time Factor)

- **定义**: 生成 1 秒语音所需的计算时间。RTF < 1 表示实时，RTF < 0.1 表示 10 倍实时
- **注意**: 严重依赖硬件（A100 vs V100）、batch size、序列长度
- **典型值**: AR 模型 RTF 0.3-1.0；NAR 模型 RTF 0.01-0.1

#### 首包延迟 (First-packet Latency)

- **定义**: 从输入文本到开始输出第一个音频帧的时间
- **对流式系统关键**: [[CosyVoice2]] ~200ms、[[GLM-TTS]] ~300ms
- **包含**: 文本前端处理 + 第一个 chunk 的模型推理时间

## 主要 Benchmark 与数据集

### 通用评测

| Benchmark | 来源 | 规模 | 特点 |
|-----------|------|------|------|
| LJSpeech test | 公开 | ~100 句 | 单说话人英文，最老的标准集 |
| LibriTTS test-clean | 公开 | ~500 句 | 多说话人英文，清洁录音 |
| LibriTTS test-other | 公开 | ~500 句 | 多说话人英文，嘈杂录音 |
| Seed-TTS-eval | 字节 | 中英各 ~100 句 | 困难样本、标准化流程 |
| [[EmergentTTS-Eval]] | 学术 | ~200 句 | 长文本/数字/特殊格式鲁棒性 |
| [[TTSDS2]] | 学术 | 框架 | 多维度自动化评测框架 |

### 专项评测

| 评测 | 测什么 | 代表使用 |
|------|--------|---------|
| [[SUPERB]] | 语音通用任务（含 TTS 相关） | 语音 SSL 模型评测 |
| [[Dynamic-SUPERB]] | 动态新增任务 | 语音 LLM 评测 |
| [[SpeechJudge]] | LLM-as-judge 自动评测 | 避免人工 MOS 的替代方案 |
| [[Emilia]] | 大规模 TTS 数据集 + 评测 | 开源 TTS 系统标准化训练和评测 |

## 各系统评测对比矩阵

> 以下数据均来自各论文自报，**不同论文的评测设置不完全可比**。仅作参考。

### 英文零样本 TTS

| 系统 | WER↓ | SIM-O↑ | MOS↑ | 评测集 |
|------|------|--------|------|--------|
| [[VALL-E]] | 5.9% | 0.580 | 3.8 | LibriSpeech |
| [[NaturalSpeech2]] | - | 0.612 | 4.2 | LibriSpeech |
| [[CosyVoice]] | 4.3% | 0.735 | 4.1 | Seed-TTS-eval |
| [[CosyVoice2]] | 2.5% | 0.778 | - | Seed-TTS-eval |
| [[F5-TTS]] | 2.4% | 0.682 | - | Seed-TTS-eval |
| [[IndexTTS2]] | 2.1% | 0.701 | - | Seed-TTS-eval |

### 中文零样本 TTS

| 系统 | CER↓ | SIM-O↑ | 评测集 |
|------|------|--------|--------|
| [[CosyVoice]] | 3.2% | 0.758 | Seed-TTS-eval zh |
| [[CosyVoice2]] | 1.8% | 0.796 | Seed-TTS-eval zh |
| [[CosyVoice3]] | 0.95% | 0.805 | Seed-TTS-eval zh |
| [[IndexTTS2]] | 1.3% | 0.812 | Seed-TTS-eval zh |

## 评测陷阱与红旗

### 常见不可靠做法

1. **Cherry-pick baseline**: 只和弱基线比，避开同期强竞品
2. **不同评测集**: A 系统报 LibriSpeech 结果，B 系统报 Seed-TTS-eval 结果，直接对比无意义
3. **MOS 评分膨胀**: 不同评审池、不同评分指导导致 MOS 绝对值不可跨论文比较
4. **只报最好指标**: 报 WER 低但不报 SIM-O，或反之
5. **缺少显著性检验**: MOS 差 0.1 是否有统计显著性？多数论文不报 p-value
6. **测试集泄露**: 用 LibriSpeech 的数据同时训练和测试

### 如何读懂一篇论文的评测

1. 看**评测集是否标准化**（Seed-TTS-eval > 自建集）
2. 看**baseline 是否公平**（同数据量、同模型大小？）
3. 看**是否报了多个维度**（WER + SIM-O + MOS 三者缺一不可）
4. 看**是否有 ablation**（证明每个模块的贡献）
5. 看**数字的量级**（WER 从 3% 降到 2.5% vs 从 10% 降到 5%，意义完全不同）

## 评测工具推荐

| 工具 | 用途 | 获取方式 |
|------|------|---------|
| Whisper large-v3 | WER/CER 计算 | `pip install openai-whisper` |
| WavLM-TDNN | SIM-O 计算 | SpeechBrain |
| UTMOS | 自动 MOS | HuggingFace |
| pesq / polqa | 客观语音质量 | ITU 标准实现 |
| pysepm / pysptk | 频谱分析 | pip |

---

*最后更新: 2026-05-25*
