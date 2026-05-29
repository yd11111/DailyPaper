---
title: "Comprehensive Benchmarking of Long-Form Speech Generation in Diverse Scenarios"
method_name: "SwanBench-Speech"
authors: [Changhao Pan, Rui Yang, Han Wang, Zhuan Zhou, Xuming He, Wenxiang Guo, Ziyue Jiang, Ruiqi Li, Yu Zhang, Chenyuhao Wen, Ke Lei, Xiang Yin, Jingyu Lu, Zhiyuan Zhu, Zhou Zhao]
year: 2026
venue: ACL 2026 Findings
arxiv_id: "2605.28618"
tags: [tts, benchmark, evaluation, long-form-tts, dialogue-tts]
zotero_collection:

# === 论文核心技术元数据（benchmark 论文，多数维度 N/A）===
lm_init: "N/A — benchmark 论文，无新模型训练"
training_loss: "N/A"
tokenizer_arch: "N/A"
multitask: false
training_data: "评测集 1,101 样本：在线文本（audiobooks/drama/news）+ 在线音频（YouTube/Bilibili/Spotify/RedNote/Apple Podcasts，先 ZipEnhancer 去噪 + DNS-MOS≥3.5 + 3D-Speaker 切分 + SenseVoice 转写 + 人工校对）+ GPT-5 生成 [已 verify §3.2 + §A.2]"
post_training: "N/A"
codec_detail: "N/A"
benchmark_metrics: "7 维：Timbre Consistency (WavLM-TDCNN, 3s window/2s stride 内余弦平均, ↑) / Reverb Consistency (SRMR std dev with VAD filter, ↓) / Sound Fidelity (SQUIM-PESQ reference-free, ↑) / Content Accuracy (FunASR-Nano WER/CER, ↓) / Prosodic Coherence (SpeechJudge 基于 Qwen2.5-Omni-7B 的 1-5 分, ↑) / Expressive Richness (Gemini3-Pro LALM 评 10s chunk 平均, ↑) / Expressive Hierarchy (Gemini3-Pro LALM 评整段, ↑) [已 verify §3.4 + §C.1-C.7]"
benchmark_size: "1,101 样本 = 380 (Acoustics 34.5%) + 339 (Semantics) + 382 (Expressiveness)；ZH:EN = 49.3%:50.7%；single/dual/multi-speaker 全覆盖（multi=101 条 3-4 说话人对话）[已 verify §B.1 + Fig.8]"
human_alignment_srcc: "Timbre SRCC=0.75 PLCC=0.77 / Prosody SRCC=0.82 / Richness SRCC=0.71 / Hierarchy SRCC=0.62（Gemini3-Pro vs 10-rater MOS）[已 verify §3.5 + §D.1-D.4]"

# === 知识地图联动 ===
domain: TTS
subdomain: evaluation
routes: ["其他: long-form-eval-benchmark"]
problems: [evaluation, long-form-stability, prosody-control, emotion-style-control]
representations: []
related_maps:
  - "[[TTS-评测体系]]"
  - "[[TTS-核心挑战]]"
  - "[[TTS-代表模型谱系]]"
  - "[[TTS-趋势判断]]"
related_surveys:
  - "[[TTS-11篇综述综合-2026-05]]"
evidence_level: medium
maturity: emerging
last_repositioned: 2026-05-29

# === 回流状态 ===
map_backfilled: false
backfilled_at:

# === 资源本地化路径 ===
pdf_local: "~/DailyPaper/.cache/papers/2605.28618/paper.pdf"
html_local: "~/DailyPaper/.cache/papers/2605.28618/paper.html"
figures_dir: "_resources/2605.28618/figures"
github_local:
cached_at: 2026-05-29

# === 通用元数据 ===
image_source: online
arxiv_html: https://arxiv.org/html/2605.28618v1
created: 2026-05-29
---

# 论文笔记：Comprehensive Benchmarking of Long-Form Speech Generation in Diverse Scenarios

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | 浙江大学（Zhou Zhao 组主导） + 字节跳动 Seed（Xiang Yin） |
| 日期 | 2026 年 5 月 |
| 项目主页 | 论文声明"will be released on Hugging Face under CC BY-NC-SA 4.0 + codebase on GitHub"，**目前 URL 未公开**（截至 2026-05-29） |
| 评测对象 | 单说话人长篇 16 模型（10 开源 + 6 闭源）+ 对话 10 模型（6 开源 + 4 闭源） |
| 链接 | [arXiv](https://arxiv.org/abs/2605.28618) / [HTML](https://arxiv.org/html/2605.28618v1) / Code: 待公开 / HF: 待公开 |

---

## 一句话总结

> 面向长篇 TTS（含对话）的 benchmark：1,101 样本 × 17 场景 × 3 大挑战（Acoustics / Semantics / Expressiveness）× 7 自动化指标，首次系统性给出"长篇生成模型距离真人录音还差多少"的量化答案，并指出 NAR 鲁棒/AR 表达性的二元路线已不够用。

---

## 核心贡献

1. **覆盖 17 个真实场景的长篇 TTS benchmark**：把长篇生成拆成 Acoustics（6 场景：customer service / podcast / chat / debate / audiobook / interview）+ Semantics（5 场景：lesson / popular science / presentation / seminar / news）+ Expressiveness（6 场景：drama / talk show / hosting / speech / live streaming / sportscast）三大挑战。1,101 条样本，ZH:EN 严格 49.3%:50.7%，含 101 条 3-4 说话人对话子集 [已 verify §3.1 + §B.1]。
2. **7 维自动化评测协议 + 人工对齐验证**：超越传统 WER/SIM/MOS，新增 Reverb Consistency（[[SRMR]] 标准差）/ Timbre Consistency（窗口内 [[WavLM]] 嵌入余弦平均）/ Prosodic Coherence（[[SpeechJudge]] 1-5 分）/ Expressive Richness（[[Gemini 3 Pro]] LALM 评 10s 段平均）/ Expressive Hierarchy（LALM 评整段）。**与人工 MOS 的 SRCC 分别达 0.75 / 0.82 / 0.71 / 0.62**，证明 LALM-as-judge 可作为长篇主观评测的可扩展 proxy [已 verify §3.4 + §D]。
3. **20+ 主流模型的横向 leaderboard**：单说话人 = VibeVoice（开源最强 Richness 3.71 / Hierarchy 3.34）+ Minimax-Speech-02-hd（闭源最强 3.80 / 3.26）；对话 = SoulX-Podcast（开源最强 Richness 3.44 / Hierarchy 3.71）+ Gemini-2.5-pro（闭源最强 4.06 / 4.02）。**与真人录音的差距：闭源平均落后 0.93 分 Richness、0.93 分 Hierarchy；开源差距更大**[已 verify §4 + Tab.2 + Tab.3]。
4. **AR vs NAR 的"二元已不够"判断**：实测显示 NAR（[[F5-TTS]] / [[ZipVoice]]）随长度延展鲁棒（Content Accuracy 不掉），但 Expressive Hierarchy 显著低于 AR；AR（[[SparkTTS]] / [[MoonCast]]）表达性强但长度增长后 CER 急剧上升（SparkTTS CER 32.9%）。**推荐 Coarse-to-Fine 混合范式**（[[NaturalSpeech 3]] / SoundStorm 类）作为未来方向 [已 verify §5.2 + Fig.4]。

---

## 问题背景

### 要解决的问题

长篇（minute-level，> 100 词）TTS / 对话生成的**系统性评测**长期缺失：现有 benchmark 几乎都是句子级，长文本上 [[WER]] 已饱和（SOTA 系统 WER < 5%），无法区分模型差异，更不能反映长上下文里的"音色漂移""混响漂移""韵律连贯""情感动态"等真实问题。

### 现有方法的局限

[已 verify §1 + §2]：

| 类别 | 代表 | 局限 |
|---|---|---|
| 信号级指标 | PESQ / STOI | 句子级，长文本不适用 |
| MOS 预测网络 | UTMOS / DNSMOS / SQUIM-MOS | 训练集偏短句、偏自然度，对**表达性**预测差（§D.4 显示 traditional MOS 网络与人工 SRCC 极低） |
| 分布级 | TTSDS / TTSDS2 | 整体分布对齐，看不出单条 sample 的具体缺陷 |
| 准确率类 | WER / CER | 现代 TTS 已饱和（< 3% 常见） |
| MLLM-as-judge | EmergentTTS-Eval / MLLM-as-a-Judge | 多为粗粒度 A/B 偏好，无量化分数，且 overlook consistency |
| 长文本子集 | MinutesSpeech / LibriSpeech-Long | 场景过窄（多为 audiobook） |
| 对话评测 | SD-Eval | 同样过于聚焦个别维度 |

### 本文的动机

要让长篇 TTS 评测"既覆盖足够多场景，又能在 consistency / hierarchy 这种长文本独有维度上给出可量化、与人工对齐的分数"，必须同时做三件事：(1) 重新设计场景分类（按 Acoustic/Semantic/Expressive 拆分场景）；(2) 引入新指标（如 Reverb std dev、segment-level timbre cosine、LALM-as-judge）；(3) 全程做人工对齐验证 + 多 evaluator benchmarking [已 verify §1 末段]。

---

## 方法详解

### 领域定位

SwanBench-Speech 属于**长篇 TTS 评测 benchmark**（与 [[EmergentTTS-Eval]] / [[TTSDS2]] / [[SpeechJudge]] / MinutesSpeech 同类），但相对已有工作有三个差异化定位：(a) **场景维度按"音学挑战 → 语义挑战 → 表达挑战"重新拆分**而非按内容主题（这是受 EmergentTTS 启发但更系统化）；(b) **同时覆盖单说话人长篇 + 多说话人对话**（先前 benchmark 一般只覆盖其一）；(c) **首次把 LALM-as-judge 作为表达性主指标**（Gemini-3-Pro 与人工 SRCC 0.71 / 0.62），同时保留传统信号指标避免回归"全看 MLLM"的风险。论文同时也是 [[Zhou Zhao 组]]（浙大）一系列长篇/对话 TTS 评测工作的延续，与 [[SpeechJudge]] / [[WavRAG]] 同条研究线。

### 数据集构造流程

[已 verify §3.2 + §3.3 + §A.2 + §A.3]：

**三源采集**：

1. **在线文本语料**（audiobook / news / drama / host）：爬取 + clean-text 库清洗 + 人工 proofread + 加 speaker label
2. **在线音频媒体**（主要来源）：YouTube / Bilibili / Spotify / RedNote / Apple Podcasts → [[ZipEnhancer]] 去噪 → [[DNSMOS]] ≥ 3.5 过滤 → [[3D-Speaker]] diarization → [[SenseVoice]]-Small 转写 → 人工校对
3. **LLM 生成**（chat / presentation / customer service 等难爬取的场景）：GPT-5 + 结构化 prompt（含 scenario / topic / task），所有生成样本人工核验

**四阶段精炼**：

1. **语义去重**：GPT-5 抽 topic+keyword+summary → 用 [[SentenceBERT]] (all-MiniLM-L6-v2) 编码 → cosine > 0.8 视为冗余去除
2. **质量过滤**：GPT-5 在 1-5 分制评 expression clarity + content coherence，< 2 分扔掉
3. **隐私伦理过滤**：[[DeepSeek V3.2]] + CoT 两步走（(a) PII 匿名化：私人姓名替换、公众人物保留；(b) ethics check：仇恨/暴力/性暗示/偏见过滤）
4. **人工 review**（3 阶段）：占位符回填真实但虚构实体 / 残余错误清理 / 数据补齐

最终得 **1,101 条样本**，按场景与挑战类别均衡分布（Acoustics 34.5% 略多于其他两类）。

### 7 维评测指标设计

[已 verify §3.4 + §C.1-C.7]：

#### Acoustics（3 指标）

| 指标 | 计算方法 | 方向 |
|---|---|---|
| **[[Timbre Consistency]]** | 3s window / 2s stride，提取 [[WavLM]]-TDCNN speaker embedding 序列；对所有 distinct pair 计算 cosine sim 平均；多说话人时按 forced alignment 分流后按说话人平均（Paraformer 中 / WhisperX 英） | ↑ |
| **[[Reverb Consistency]]** | 3s/2s window 计算 [[SRMR]]（speech-to-reverberation modulation energy ratio），用 VAD 过滤 > 60% 静音的窗口；取序列**标准差** | ↓（std 越小越稳定） |
| **[[Sound Fidelity]]** | [[SQUIM-PESQ]]（reference-free PESQ，via Torchaudio），范围 -0.5 ~ 4.5 | ↑ |

#### Semantics（2 指标）

| 指标 | 计算方法 |
|---|---|
| **[[Content Accuracy]]** | [[FunASR-Nano]] 转写（论文标该 ASR 在 LibriSpeech-clean WER=1.76%、Fleurs-zh CER=2.56%）→ 双向标准化（去标点 / 标准化空格 / 繁→简、过滤非 ASCII） → JiWER 算 WER（英）/ CER（中） |
| **[[Prosodic Coherence]]** | [[SpeechJudge]]（基于 Qwen2.5-Omni-7B 微调，专为韵律评测）打 1-5 分；论文额外 refine prompt 强化"长上下文韵律敏感性"；每条做 10 次独立评分取均值 |

#### Expressiveness（2 指标）

| 指标 | 计算方法 |
|---|---|
| **[[Expressive Richness]]** | 把 audio 切 10s chunk → [[Gemini 3 Pro]] LALM 用 prompt 评每 chunk 的 emotional resonance / character portrayal / storytelling → 取算术平均 |
| **[[Expressive Hierarchy]]** | **整段（不切片）**喂给 Gemini-3-Pro → 评 emotional variation / vocal dynamics / scene appropriateness 1-5 分 |

**为何选 Gemini-3-Pro 当 LALM judge**：§D.4 benchmark 了 4 个 MOS 预测网（UTMOS / UTMOSv2 / SQUIM-MOS / DNSMOS）+ 8 个 LALM（GPT-4o / Qwen3-Omni-Instruct/Flash / StepFun-Audio-R1 / Gemini-2.5-flash/pro / Gemini-3-flash/pro），**只有 Gemini-3-Pro 在 Expressive Richness/Hierarchy 上与 10-rater MOS 对齐分别达 SRCC 0.71 / 0.62**，且 5 次独立评分仅 11 个样本不一致（接近人类 evaluator 的稳定性）。**传统 MOS 网络与表达性 MOS 几乎不相关**——侧面说明 MOS 训练集普遍缺乏 expressive label。

### 人工对齐验证（§3.5 + §D）

招 10 名专家听众（5 男 5 女，含工业音频工程师、直播专家、语音处理研究者），共花费 $2,000：

- **Timbre**：50 样本 MOS → SRCC=0.75 / PLCC=0.77 / KRCC=0.59；额外给出 **score 阈值经验**：< 0.85 = 显著音色漂移；0.85-0.90 = 轻微 mutation；≥ 0.93 = 接近 ground truth
- **Sound Fidelity**：50 样本（中文+长篇 generalization test）→ SRCC=0.72 / PLCC=0.47
- **Prosodic Coherence**：50 对样本 -2~+2 偏好评 → SRCC=0.82，差异 > 1 分等于"明显感知差异"
- **Expressive Richness**：200 样本 → Gemini-3-Pro SRCC=0.71（在 12 个 evaluator 中最高）
- **Expressive Hierarchy**：同上集 → SRCC=0.62
- **Inter-rater correlation**：高（具体值在 Tab.6），证明 evaluator 协议可靠

---

## 关键公式

### 公式1: [[Timbre Consistency|窗口对说话人嵌入余弦相似度]]

$$
\mathrm{sim}_{i,j}=\cos\!\left(\frac{\mathbf{e}_{i}}{\lVert\mathbf{e}_{i}\rVert},\frac{\mathbf{e}_{j}}{\lVert\mathbf{e}_{j}\rVert}\right),\quad\forall i\neq j
$$

**含义**：对单说话人长篇音频，把 3s window / 2s stride 提取的所有 [[WavLM]] speaker embedding 两两计算 cosine，取**平均**作为该段的 timbre consistency 分数。

**符号说明**：
- $\mathbf{e}_i$：第 $i$ 个 3 秒窗口经 WavLM-TDCNN 提取的 speaker embedding
- $n$：窗口数（论文 §C.1 解释 3s 窗口选择是因为 speaker verification 模型一般针对 2-4s 优化）

### 公式2: [[Multi-speaker Timbre|多说话人 Timbre]] 平均

$$
\mathrm{Score}_{\text{multi}}=\frac{1}{K}\sum_{k=1}^{K}a_{k}
$$

**含义**：对话场景下，先 forced alignment 切出 $K$ 个 speaker 各自的语音流 $\tilde{w}_k$，每个 stream 内按公式 1 算 timbre 一致性得 $a_k$，再对 $K$ 说话人取平均。

**符号说明**：
- $K$：3D-Speaker 验证后的说话人数
- $a_k$：第 $k$ 个 speaker 自身的 timbre consistency

### 公式3: [[Expressive Richness]] 段平均

$$
\mathrm{Score}_{\text{rich}}=\frac{1}{M}\sum_{i=1}^{M}s_{i}
$$

**含义**：把音频切成 $M$ 个**互不重叠的** 10 秒段 $\{c_i\}$，每段用 Gemini-3-Pro 评 expressiveness 分 $s_i$（1-5），取算术平均。10s 选定是因为对齐"chunk-based 长篇合成的典型生成长度"，避免 inter-chunk 不一致干扰评测。

### 公式4: [[Real Time Factor|RTF]] 推理速度

$$
\mathrm{RTF}=\frac{T_{\text{inference}}}{T_{\text{audio}}}
$$

**含义**：经典指标，论文 §F.1 表 9-10 报告各模型 RTF；§5.2 结论是"NAR 模型显著快于 AR，符合并行解码理论预期"。

### 公式5: [[Human Preference Score|人工偏好分]]

$$
\mathcal{S}_{\text{pref}}(A,B)=\frac{1}{N}\sum_{i=1}^{N}s_i
$$

**含义**：Prosody 验证时用，10 个 rater 在 [-2, 2] 上给 A 相对 B 的偏好分，取平均；与本论文 Prosodic Coherence 分数差异之间的 SRCC = 0.82。

---

## 关键图表

### Figure 1: SwanBench-Speech 概览

**论文 Figure 1 在 arXiv HTML 中无 img 资源**（仅是带 caption 的文本/图标布局）。语义内容：左侧展示三大挑战 + 17 场景的样本分布；右侧示意评测协议。可参考 PDF 第 1 页或本地缓存的 `_resources/2605.28618/figures/fig-000.png` 类似的截图。

### Figure 2: 数据集构造与精炼流程

![Figure 2](https://arxiv.org/html/2605.28618v1/x2.png)

**说明**：四阶段构造管线——(1) 按 Acoustic/Semantic/Expressive 三大挑战立纲；(2) 选 17 场景；(3) 三源采集（在线文本 / 在线音频 / GPT-5 生成）；(4) 四步精炼（SentenceBERT 去重 / GPT-5 质量 / DeepSeek-V3.2 伦理 / 人工 review）。

### Figure 3: 三大挑战上的雷达对比

![Figure 3](https://arxiv.org/html/2605.28618v1/x3.png)

**说明**：将所有评测结果归一化到 1-5 区间后画三张雷达图（每挑战一张），可视化各模型在该挑战内 7 维指标的表现。**关键观察**：表达性场景下几乎所有模型都出现退化，特别是 Expressive Richness——反直觉，因为表达性场景本应是表达力的上限发挥地。作者归因为"模型缺乏对表达性数据的有效训练"。

### Figure 4: 内容准确率随生成长度的退化

![Figure 4](https://arxiv.org/html/2605.28618v1/x4.png)

**说明**：横轴为文本句子数。**核心 finding**：[[SparkTTS]] 的 Content Accuracy 随长度增长**剧烈退化**（训练集 VoxBox 平均段长 < 10s 导致 short-form bias）；NAR 模型（F5-TTS / ZipVoice）保持稳定。也是论文支撑"AR 错误累积 / NAR 鲁棒"判断的最直接证据。

### Figure 8: SwanBench-Speech 的五维统计

![Figure 8](https://arxiv.org/html/2605.28618v1/x5.png)

**说明**：语言（ZH 49.3% / EN 50.7%）/ 说话人数（single / dual / multi）/ 三大挑战占比 / 17 场景占比 / 话题词云。

### Figure 9: 文本长度分布

![Figure 9](https://arxiv.org/html/2605.28618v1/x6.png)

**说明**：中文按字数、英文按词数。**关键设计选择**：平均长度 271.8（中） / 174.6（英），范围 80-500，对应"分钟级合成"。作者承认 audiobook 实际可超过 10 分钟，但选择这个范围是因为"100+ 词已足够暴露长依赖问题"（[[FireRedTTS]]、[[ISDrama]]、[[SoulX-Podcast]] 等先前工作也支持这一点）。

### Figure 10: 多模型在长度延展下的全维度退化曲线

![Figure 10](https://arxiv.org/html/2605.28618v1/x7.png)

**说明**：扩展 Figure 4，跟踪 Reverb / Prosodic / Hierarchy / Timbre Similarity / Timbre Consistency 等多维度随长度变化。**结论**：(1) Reverb Consistency / Prosodic Coherence / Expressive Hierarchy 退化最显著——证实长依赖问题；(2) Timbre Similarity / Consistency 相对稳定——in-context learning 范式 ([[CosyVoice 2]] / [[MegaTTS3]]) 在说话人保持上仍有效；(3) Content Accuracy 大多稳定（除 SparkTTS 例外）。

### Figure 11: 闭源模型在各场景的雷达

![Figure 11](https://arxiv.org/html/2605.28618v1/x8.png)

**说明**：单说话人长篇生成下闭源模型在 17 场景的归一化分布。

### Table 2: 单说话人长篇 TTS — 完整结果（[已 verify §4.2 Tab.2 原文表格]）

> 方向：Timbre/Fidelity/Prosody/Richness/Hierarchy ↑（越高越好），Reverb/CER/WER ↓（越低越好）。粗体=最佳，下划线（这里只能用文字标注）=次佳。

#### 开源模型

| Model | Timbre ↑ | Reverb ↓ | Fidelity ↑ | CER/WER ↓ | Prosody ↑ | Richness ↑ | Hierarchy ↑ |
|---|---:|---:|---:|---:|---:|---:|---:|
| CosyVoice-2 | 0.92 | 2.35 | 3.80 | 0.032 / 0.168 | 3.23 | 3.02 | 2.76 |
| CosyVoice-3 | **0.94** | 2.26 | 3.83 | 0.034 / 0.141 | 3.31 | 2.80 | 2.45 |
| FishSpeech | 0.93 | 1.79 | **4.10** | 0.043 / 0.113 | 3.80 | 2.66 | 2.90 |
| F5-TTS | 0.90 | 1.82 | 3.39 | 0.072 / 0.113 | 3.41 | 3.07 | 2.77 |
| GLM-TTS | **0.94** | **1.62** | 3.95 | 0.035 / 0.118 | 3.64 | 2.68 | 2.54 |
| IndexTTS-2 | **0.94** | 1.72 | 2.77 | 0.033 / 0.135 | 3.64 | 3.59 | 2.96 |
| MegaTTS-3 | 0.93 | 1.81 | 3.55 | 0.035 / 0.108 | 3.61 | 2.81 | 2.53 |
| SparkTTS | 0.93 | 1.79 | 3.59 | 0.329 / 0.240 | 2.58 | 3.47 | 2.38 |
| **VibeVoice** | 0.93 | 2.15 | 3.82 | 0.047 / 0.111 | **3.90** | **3.71** | **3.34** |
| ZipVoice | 0.90 | 2.06 | 3.51 | 0.072 / 0.396 | 3.19 | 2.44 | 2.11 |
| **平均** | 0.93 | 1.95 | 3.63 | 0.073 / 0.164 | 3.43 | 3.03 | 2.67 |

#### 闭源模型

| Model | Timbre ↑ | Reverb ↓ | Fidelity ↑ | CER/WER ↓ | Prosody ↑ | Richness ↑ | Hierarchy ↑ |
|---|---:|---:|---:|---:|---:|---:|---:|
| ElevenLabs Multilingual V2 | **0.96** | 3.05 | **4.02** | 0.100 / 0.115 | 3.50 | 2.33 | 2.68 |
| Gemini-2.5-pro-preview-tts | 0.91 | 1.44 | 3.16 | 0.058 / 0.169 | 3.91 | **4.14** | **3.51** |
| InWorld-TTS-1-max | 0.93 | 2.19 | 3.73 | 0.053 / 0.113 | 3.71 | 3.68 | 3.03 |
| **Minimax-Speech-02-hd** | 0.93 | **1.38** | 3.82 | **0.032** / 0.119 | **3.95** | 3.80 | 3.26 |
| OpenAI tts-1-hd | 0.92 | 1.74 | 2.68 | 0.043 / 0.119 | 3.91 | 3.46 | 3.25 |
| Seed-TTS 2 | **0.94** | 1.95 | 3.88 | 0.106 / 0.193 | 3.74 | 3.10 | 2.34 |
| **平均** | 0.93 | 1.96 | 3.55 | 0.065 / 0.138 | 3.79 | 3.42 | 3.01 |
| **Real Speech**（参考）| **0.96** | 1.91 | 3.62 | 0.070 / 0.074 | **4.04** | **4.35** | **3.94** |

**关键发现**：
- **Timbre**：开闭源平均都到 0.93，已接近真人 0.96；CosyVoice-3 / GLM-TTS / IndexTTS-2 / Seed-TTS-2 / ElevenLabs 都达到或接近 0.94+ "competent" 阈值
- **Reverb**：闭源 Minimax / Gemini-2.5-pro 反而**比真人录音更稳定**（1.38 / 1.44 vs 1.91）——可能说明合成系统的"声学环境过于干净/单一"，反而失去自然录音的合理波动
- **Content Accuracy**：[[SparkTTS]] CER 32.9%（异常高）+ [[ZipVoice]] WER 39.6% 是异常点；其他模型 CER < 10% / WER < 25%，已接近真人
- **Expressiveness**：所有模型 Richness/Hierarchy 都明显低于真人（4.35 / 3.94），最强的 [[Gemini-2.5-pro]] 才到 4.14 / 3.51；**这是当前长篇 TTS 最大的未解 gap**

### Table 3: 对话生成 — 完整结果（[已 verify §4.2 Tab.3 原文表格]）

#### 开源对话模型

| Model | Timbre ↑ | Reverb ↓ | Fidelity ↑ | CER/WER ↓ | Prosody ↑ | Richness ↑ | Hierarchy ↑ |
|---|---:|---:|---:|---:|---:|---:|---:|
| FireRedTTS-2 | 0.93 | 3.48 | 2.62 | 0.075 / 0.131 | 3.24 | 2.72 | 2.81 |
| MoonCast | 0.90 | 3.06 | 2.62 | 0.313 / 0.125 | 3.16 | 2.68 | 2.70 |
| MOSS-TTSD | 0.91 | 3.55 | 2.89 | 0.148 / 0.239 | 2.79 | 3.21 | 2.99 |
| **SoulX-Podcast** | **0.93** | 3.51 | **3.96** | **0.061 / 0.090** | **4.01** | 3.44 | **3.71** |
| VibeVoice | 0.91 | 3.59 | 3.35 | 0.106 / 0.125 | 3.57 | **3.76** | 3.37 |
| ZipVoice-Dialog | 0.91 | 3.53 | 2.66 | 0.069 / 0.114 | 3.67 | 2.62 | 2.80 |
| **平均** | 0.92 | 3.45 | 3.02 | 0.129 / 0.137 | 3.41 | 3.07 | 3.06 |

#### 闭源对话模型

| Model | Timbre ↑ | Reverb ↓ | Fidelity ↑ | CER/WER ↓ | Prosody ↑ | Richness ↑ | Hierarchy ↑ |
|---|---:|---:|---:|---:|---:|---:|---:|
| ElevenLabs Multilingual V2 | 0.93 | 4.43 | 3.48 | 0.127 / 0.109 | 3.67 | 2.84 | 3.46 |
| **Gemini-2.5-pro-preview-tts** | 0.92 | 3.17 | 3.01 | 0.086 / 0.092 | **4.06** | **4.06** | **4.02** |
| OpenAI tts-1-hd | **0.93** | 2.98 | 2.28 | 0.104 / 0.103 | 3.69 | 3.29 | 3.70 |
| SeedTTS-Podcast | 0.91 | 2.85 | **3.89** | **0.063 / 0.108** | 3.93 | 3.84 | 3.84 |
| **平均** | 0.92 | 3.36 | 3.17 | 0.095 / 0.103 | 3.83 | 3.51 | 3.76 |
| **Real Dialogue**（参考）| 0.95 | **2.73** | 2.94 | **0.050** / 0.137 | 3.95 | — | — |

**关键发现**：
- **对话场景 Reverb 普遍很差**：开源平均 3.45 / 闭源 3.36，真人 2.73——**多说话人下维持全局声学一致是核心瓶颈**（频繁说话人切换打断混响连续性）
- **SoulX-Podcast 是开源最强**，在 6 个开源对话模型里 5/7 指标领先，已逼近闭源平均
- **VibeVoice 跨长篇+对话双榜**（单说话人和对话都进 top-3 开源），是 2025-2026 开源圈最全能的长篇模型
- 对话 Real Dialogue 在 Reverb 上 2.73（**优于所有合成系统**）证明：真实录音的"自然环境一致性"是当前所有合成模型都未掌握的能力

### Table 4-5: LALM 评测器对齐分（§D.4）

11+ 个 evaluator（4 个 MOS 网 + 7+ 个 LALM）的人工对齐分排名：**Gemini-3-Pro 在 Richness/Hierarchy 双榜第一**；开源 LALM 中 Qwen3-Omni-Flash / Instruct **超越 GPT-4o**，与 Gemini-2.5-Pro 差距很小；所有传统 MOS 网络（UTMOS / UTMOSv2 / SQUIM-MOS / DNSMOS）在 expressiveness 上与人工**几乎不相关**。

### Table 13: 多说话人对话（3-4 speaker）子集结果（§F.4）

只评了 3 个支持多 speaker 的闭源（ElevenLabs / Gemini-2.5-pro / OpenAI tts-1-hd）。预示未来 multi-speaker 长篇合成的评测空间。

### Table 14-15: 中英双语分语种结果（§G.3）

**重要观察**：尽管所有模型都宣称双语，**多数在两种语言上表现差异显著**——比如 ElevenLabs Richness Chinese=1.79 / English=2.87；SeedTTS-Podcast Chinese=4.19 / English=3.49；**Gemini-2.5-pro 是唯一在双语上保持一致的模型**。

---

## 实验

### 评测对象

| 类别 | 单说话人长篇（10 开源 + 6 闭源） | 对话（6 开源 + 4 闭源） |
|---|---|---|
| 开源 | [[ZipVoice]], [[SparkTTS]], [[CosyVoice 2]] (0.5B), [[CosyVoice3]] (0.5B), [[GLM-TTS]], [[MegaTTS3]], [[IndexTTS2]], [[FishSpeech]] (1.5), [[F5-TTS]], [[VibeVoice]] | [[ZipVoice-Dialog]], [[MoonCast]], [[MOSS-TTSD]], [[FireRedTTS2]], VibeVoice, [[SoulX-Podcast]] |
| 闭源 | Gemini-2.5-pro-preview-tts, OpenAI-tts-1-hd, ElevenLabs Multilingual V2, [[Minimax-Speech 02 HD]], InWorld-TTS-1-max, [[Seed-TTS]] 2 | Gemini-2.5-pro, OpenAI-tts-1-hd, ElevenLabs Multilingual V2, [[SeedTTS-Podcast]] |

### 评测器模型

| 用途 | 模型 |
|---|---|
| Speaker embedding | [[WavLM]]-TDCNN（microsoft/UniSpeech） |
| Forced alignment | Paraformer（modelscope iic 中）+ [[WhisperX]]（英） |
| WER/CER ASR | [[FunASR-Nano]](Fun-ASR-Nano-2512, LibriSpeech-clean WER 1.76%) |
| 韵律评测 | [[SpeechJudge]]（Qwen2.5-Omni-7B fine-tune） |
| 表达性评测 | [[Gemini 3 Pro]]（with prompt enhancement） |
| 静音/去噪 | [[ZipEnhancer]] / FSMN-VAD / [[Silero-VAD]] |

### 实现细节

- **硬件**：8× RTX 4090 + Intel Xeon Gold 6530，Ubuntu 22.04
- **软件**：Python 3.10 / PyTorch 2.8.0 / Torchaudio 2.8.0 / Transformers 4.57.3
- **采样率**：所有合成 audio resample 到 **24 kHz** 评测
- **Window/Stride**：Timbre + Reverb 都用 3s window / 2s stride（§F.2 ablation 验证此为最佳）
- **Reference voice**：开源模型用 25 个来自 [[Emilia]]/[[AISHELL-3]]/[[NCSSD]]/[[LibriSpeech]]/[[MSP-Podcast]]/ChildMandarin 的 prompt（每模型选最强 voice）；闭源用各家官方 voice profile
- **特殊调整**：MegaTTS3 用 maintainer 提供的 VAE latent（VAE encoder 不开源）；IndexTTS2 关掉 `use_emo_text=false`；CosyVoice3 用官方默认 system prompt
- **预算**：数据采集 + 标注 $550；user study $2,000

### 结果可信度

| 可信度 | 结果 | 理由 |
|---|---|---|
| **高** | 7 维量化分数的相对排序、CER/WER 数字、SRMR std 数字、Real Speech vs 合成的 gap | 客观 ASR/codec/SQUIM-PESQ/SRMR 工具链可复现；FunASR-Nano 性能在 LibriSpeech-clean WER=1.76% 接近 SOTA；表 1-3 含 mean±std 而非裸数字 |
| **中** | Richness/Hierarchy 的具体分数 + 跨模型差异 | 依赖 Gemini-3-Pro 闭源 LALM（论文 §H 已自承"Dependency on Closed-source Models"是 limitation）；不同 prompt 设计可能改变绝对分；SRCC 0.71/0.62 显示与人工有一定相关但 < 0.8 |
| **中** | "VibeVoice 是开源最强 / SoulX-Podcast 是对话开源最强" | 仅 2026-05 时间窗的快照，且每模型选了 best voice prompt，可能有过拟合 prompt 的隐患（论文 §H "Timbre Sensitivity" 已自承） |
| **低** | "AR vs NAR 二元已不够 → Coarse-to-Fine" 的论断 | 这是基于"F5-TTS hierarchy 低 / SparkTTS CER 高"的两点归纳，证据数量小；NaturalSpeech 3 类 coarse-to-fine 并未在本 benchmark 上正面证明优于纯 AR/NAR |
| **低** | "Reverb std 越低越好"的隐含价值取向 | 论文 §C.2 自承 Outdoor Live Streaming 类场景本就需要 dynamic acoustic shifts，"reverb std 低 = 好"在这些场景下可能反而违反真实感（Minimax Reverb 1.38 比真人 1.91 还"低"也支持这点）|

---

## 批判性思考

### 核心 Claim 审查

1. **Paper Claim**：SwanBench-Speech 提供"comprehensive automatic metrics aligned with humans"。
   **My Assessment** [基于 §3.5 + §D 已读]：Prosody SRCC=0.82 / Timbre SRCC=0.75 在长篇 TTS 评测里属较高水准，但 Expressive Hierarchy SRCC=0.62 仍偏弱——意味着该指标的相对排名可信，但分数间的细微差异（< 0.2）可能不反映人类感受。"comprehensive" 在覆盖度上成立，"aligned with humans" 在 Prosody/Timbre 上成立但在 Hierarchy 上要打折扣。

2. **Paper Claim**："SOTA open-source models already match or even surpass the best proprietary systems on several evaluation dimensions"。
   **My Assessment**：在 Timbre Consistency 上确实成立（CosyVoice3/GLM-TTS/IndexTTS2 等开源 0.94 ≥ 闭源最佳 0.94）；但**整体上闭源仍领先**（Richness 闭源平均 3.42 vs 开源 3.03，Hierarchy 3.01 vs 2.67）。论文措辞合理但读者容易过度乐观。

3. **Paper Claim**：长篇 TTS 应走 "Coarse-to-Fine Architecture" 替代 AR/NAR 二元。
   **My Assessment**：论据仅基于 F5-TTS Hierarchy 低 + SparkTTS CER 32.9% 这两个点。然而**最强开源 VibeVoice 本质是 NAR diffusion** [基于 [[VibeVoice]] 笔记记忆，未深核 verify]，**最强闭源 Gemini-2.5-pro 架构未公开**。直接得出"Coarse-to-Fine"是正解的证据链有跳跃——它更像是一个 well-motivated speculation 而非 benchmark 直接支撑的结论。

4. **Paper Claim**："The 100+ word range is sufficient to reveal long-term dependency issues."
   **My Assessment** [已 verify §B.2]：合理，论文确实通过 Figure 4/10 实证 100+ 词后多维度退化。但**真正的长篇场景如有声书需要 10+ 分钟**——本 benchmark 在 minute-level 与 hour-level 之间留下 gap，未来扩展空间大。

### 优点

1. **场景设计有人工智识深度**：按"音学/语义/表达"挑战拆分场景而非按内容主题，是这套 benchmark 最有思想的部分；17 场景对应真实下游应用，比 LibriSpeech-Long 的"audiobook only"广得多。
2. **Reverb Consistency 作为新指标设计精妙**：把 SRMR 用作"时间稳定性"度量而非绝对值——这恰好规避了"不同录音环境差异 vs 同一录音内的漂移"的混淆，是评估长篇音频"声学场是否漂移"的优雅做法。
3. **LALM-as-judge benchmark 做得彻底**：§D.4 比较 12 个 evaluator + 5 次重复评测稳定性测试，比单纯"用 GPT-4o 评一下"严肃得多。结论"传统 MOS 网络在表达性上几乎无效"是重要的领域校正。
4. **同时 cover 单说话人 + 对话** + **101 条 multi-speaker 子集**，未来支持 multi-talker 工作扩展。
5. **完整 ablation 链**（§F.2 window size + §F.3 length + §F.4 multi-speaker）+ 真人对齐验证 + inter-rater correlation——评测论文该有的严肃性都做到了。
6. **数据采集流程透明**（§A.2 列了具体爬取源 + 三阶段人工 review），并明确给了标注成本 ($550 + $2000)；CC BY-NC-SA 4.0 许可证负责任。

### 局限性

1. **依赖闭源 LALM 评测器**（论文 §H 自承）：Richness/Hierarchy 严重依赖 Gemini-3-Pro；当 Google 升级 API 时**所有过去分数失去可比性**。论文承诺"未来 distill 开源 evaluator"，但目前是真实 reproducibility 风险。
2. **代码 + benchmark 当前未公开**（截至 2026-05-29）：论文只承诺"will be released"——这意味着外部研究者**目前无法跑评测**，结论尚不能被独立验证（这是 evidence_level 标 medium 而非 high 的关键原因）。
3. **Prompt voice 仅 20+ speaker**（§H 自承）：开源模型在 prompt voice 选择上敏感性大，"per model 选 best voice" 的策略可能引入 prompt 过拟合偏差。
4. **长度上限 100-400 词** = minute-level，远不到 hour-level audiobook 实际需求；论文承认但未解决。
5. **Reverb std "越低越好"的价值取向**在 outdoor / live streaming 等场景下反真实（Minimax 1.38 比真人 1.91 还低，可能说明"过于干净"而非"更好"）——论文 §C.2 自承但未在主指标中调整。
6. **语言只 ZH + EN**，低资源语言 + 方言完全未覆盖。
7. **表达性场景集（drama / sportscast / live streaming）**的数据获取最难，可能存在选样偏差——爬取的"表达性"样本未必代表该场景全分布。
8. **某些技术决策的合理性缺论证**：例如为什么 Richness 用 10s chunk 平均而 Hierarchy 用整段？论文 §3.4 解释是"chunk align with 典型生成长度"，但**没有 ablation 实证 10s vs 5s vs 20s 的影响**。

### 潜在改进方向

1. **开源 distilled LALM evaluator**：把 Gemini-3-Pro 的判断蒸馏到 7B 级开源 LALM，解锁 reproducibility（论文 §H 已自列为 future work）
2. **扩展到 hour-level**：增加 5 / 10 / 30 / 60 分钟梯度的样本子集，研究**真正长依赖**问题
3. **细分 Reverb 评测**：区分"室内场景应稳定"vs"户外场景应允许漂移"，给不同场景配不同 reverb 价值取向
4. **Multi-lingual 扩展**：增加西/法/日/阿拉伯/方言子集，benchmark 多语种长篇能力
5. **加入 instruction-following 维度**：响应 §H 自列方向，引入 InstructTTSEval 类的长篇 instruction 跟随能力评测
6. **Multi-speaker 子集扩到 3-4 speaker 以上**：当前 multi-speaker 子集只有 101 样本，是该论文最薄的部分

### 可复现性评估

- [ ] 代码开源（论文承诺但未公开）
- [ ] 测试集开源（论文承诺 HF 发布但未公开）
- [x] 评测协议详细（§3.4 + §C.1-C.7 + §D 完整描述了每个指标的计算方法 + 参数 + 工具链）
- [x] 评测器模型公开可用（WavLM-TDCNN / FunASR-Nano / SpeechJudge 都在 HF/GitHub）
- [ ] Gemini-3-Pro 评测依赖闭源 API（核心 reproducibility 风险）
- [x] 真人对齐验证流程透明（§D + Table 4-6）
- [x] 评测对象明确版本号（CosyVoice2-0.5B / CosyVoice3-0.5B / FishSpeech-1.5 等明确）

---

## 🗺️ 在知识地图中的定位

- **所属领域**：[[TTS-领域总览]]，[[12-数据集与评测]]
- **技术路线**：N/A（benchmark 论文不属技术路线分类）；但其评测对象覆盖几乎所有 2024-2026 主流路线（codec LM / NAR Flow Matching / diffusion / 闭源 unknown）
- **核心问题**：[[TTS-核心挑战]] §6 评估方法论 + §2 长文本稳定性 + §2 情感与韵律控制 ；[[TTS-评测体系]] §长篇评测、§LALM-as-judge
- **表示层位置**：N/A（benchmark）
- **在 SpeechLM/对话框架内的位置**：[[TTS-SpeechLM-Dialogue关系]] 位置 ④ 对话生成评测（评测了 SoulX-Podcast / MoonCast / MOSS-TTSD / FireRedTTS2 / VibeVoice / SeedTTS-Podcast 等对话路线）
- **相邻工作**：
  - [[EmergentTTS-Eval]]（启发本论文的 LALM-as-judge 方法）
  - [[SpeechJudge]]（被本论文用作 Prosody Coherence 评测器）
  - [[TTSDS2]]（分布级评测，本论文的对照）
  - [[NaturalVoices]] / [[Dynamic-SUPERB]]（同类 benchmark）
  - 所有被评测模型：[[VibeVoice]]、[[SoulX-Podcast]]、[[CosyVoice 2]]、[[CosyVoice3]]、[[F5-TTS]]、[[Spark-TTS]]、[[MegaTTS3]]、[[IndexTTS2]]、[[GLM-TTS]]、[[MoonCast]]、[[MOSS-TTSD]]、[[FireRedTTS2]]、[[ZipVoice-Dialog]]、[[Minimax-Speech 02 HD]]、[[Seed-TTS]] 等

---

## 🔄 后续重估

- **2026-05-29**：初读。从评测设计 + 实证 leaderboard 两个维度看都是 2026 上半年最重要的长篇 TTS 评测工作之一，**值得作为后续读 [[VibeVoice]] / [[SoulX-Podcast]] / [[Minimax-Speech 02 HD]] 等具体模型时的对照基准**。当前 evidence_level=medium 是因为：(1) 代码/数据未发布；(2) Gemini-3-Pro 依赖；(3) 仅单团队产出，无第三方复现。一旦发布且有第三方独立 run，可上调到 high。

---

## 关联笔记

### 基于

- [[EmergentTTS-Eval]]：LALM-as-judge 方法的直接启发
- [[SpeechJudge]]：本论文 Prosody 评测器
- [[TTSDS2]]：上一代分布级评测，本论文意在补足"single-sample 细粒度评测"的 gap

### 对比

- [[NaturalVoices]]：同期长篇 TTS 数据集类工作
- [[ALLD]]：长篇对话评测
- [[CV3-Eval]]：CosyVoice3 自家发布的评测集，与本 benchmark 评测维度差异（CV3-Eval 更偏 in-the-wild generalization，SwanBench 更偏 scenario-level decomposition）

### 方法相关

- [[SRMR]]：Reverb Consistency 的核心信号（待新建概念）
- [[SQUIM-PESQ]]：Sound Fidelity 的 reference-free PESQ 实现（待新建概念）
- [[FunASR-Nano]]：Content Accuracy 的 ASR 评测器（待新建概念）
- [[Gemini 3 Pro]]：Expressive 评测的 LALM judge（待新建概念）
- [[WavLM]]：Timbre embedding 提取
- [[3D-Speaker]]：speaker diarization
- [[WhisperX]] / [[Paraformer]]：forced alignment
- [[ZipEnhancer]]：数据预处理去噪

### 评测对象（重点模型）

- [[VibeVoice]]：单说话人长篇开源最强
- [[SoulX-Podcast]]：对话开源最强
- [[CosyVoice3]]：本 benchmark 中 reverb 表现倒数（in-the-wild 训练副作用案例）
- [[SparkTTS]]：本 benchmark 中 CER 32.9% 异常（短句训练数据的"长篇退化"教训）
- [[Minimax-Speech 02 HD]] / [[Seed-TTS]] 2：闭源旗舰

---

## 速查卡片

> [!summary] SwanBench-Speech
> - **核心**：1,101 样本 × 17 场景 × 3 大挑战 × 7 自动化指标的长篇 TTS + 对话评测 benchmark
> - **方法**：把场景按 Acoustic/Semantic/Expressive 拆分；用 SRMR std + WavLM cosine + SQUIM-PESQ + FunASR + SpeechJudge + Gemini-3-Pro 组成 7 维评测；20+ 模型横向 leaderboard + 人工 SRCC 验证
> - **结果**：单说话人 VibeVoice (开) + Minimax-Speech 02 HD (闭) 最强；对话 SoulX-Podcast (开) + Gemini-2.5-pro (闭) 最强；所有模型与真人 Richness 还差 ≥0.93 / Hierarchy ≥0.93
> - **代码**：承诺 HF + GitHub 发布但截至 2026-05-29 未公开
> - **价值**：未来读长篇/对话 TTS 论文时的对照基准；LALM-as-judge 方法论的扎实参考

---

*笔记创建时间：2026-05-29*
