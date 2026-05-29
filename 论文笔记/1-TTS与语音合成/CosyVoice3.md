---
title: "CosyVoice 3: Towards In-the-wild Speech Generation via Scaling-up and Post-training"
method_name: "CosyVoice3"
authors: [Zhihao Du, Changfeng Gao, Yuxuan Wang, Fan Yu, Tianyu Zhao, Hao Wang, Xiang Lv, Hui Wang, Chongjia Ni, Xian Shi, Keyu An, Guanrou Yang, Yabin Li, Yanni Chen, Zhifu Gao, Qian Chen, Yue Gu, Mengzhe Chen, Yafeng Chen, Shiliang Zhang, Wen Wang, Jieping Ye]
year: 2025
venue: arXiv (Tech Report)
arxiv_id: "2505.17589"
tags: [tts, zero-shot-tts, codec-lm-tts, flow-matching, multilingual, post-training, industrial-tech-report]
zotero_collection: 1-TTS与语音合成

# === 论文核心技术元数据（三层 verify）===
lm_init: "warm-start from a Qwen2 LLM checkpoint via `Qwen2ForCausalLM.from_pretrained(pretrain_path)` [GitHub: cosyvoice/llm/llm.py:226-240 Qwen2Encoder; CosyVoice3LM(Qwen2LM) at L664; examples/libritts/cosyvoice3/conf/cosyvoice3.yaml:30-31]. Hidden size = llm_input_size = 896（与 Qwen2-0.5B 一致）；具体 checkpoint 路径未在 conf 写死（qwen_pretrain_path 为空占位符） [conf:12]"
training_loss: "LM 阶段为 **speech-token-only label-smoothed CE**：vocab = speech_token_size + 200 = 6761；text/instruct 位置 mask 为 IGNORE_ID 不计 loss；无文本 loss；无 KL 约束 [GitHub: cosyvoice/llm/llm.py:687-693 LabelSmoothingLoss; L302-405 prepare_lm_input_target + forward]. CFM 用 mask-only flow matching loss [conf:45 only_mask_loss=True]. GAN vocoder 用 multi-period + multi-resolution discriminator [conf:113-119]"
tokenizer_arch: "supervised multi-task FSQ tokenizer，**基于 MinMo** voice encoder (12-layer Transformer + RoPE)；FSQ 插入到 Voice Encoder₁ 之后；下游再接 Voice Encoder₂ + MinMo LLM 做五任务监督（multilingual ASR / LID / SER / AED / SA） [§2.1, paper.html L285-316]. Codebook size = (2K+1)^D = 6561（K=1, D=8） [§2.1 Eq.1-2; conf:26 speech_token_size=6561]. 推理时是 ONNX 黑盒：`speech_tokenizer_v3.batch.onnx`，**FSQ 投影维度等内部超参不可从开源 verify** [GitHub: cosyvoice/utils/onnx.py + cosyvoice/llm/llm.py:705-706]"
multitask: "在 **tokenizer 预训练阶段为 true**（5 任务 530K h 监督）[§2.1 + Tab.3]；在 **LM 训练阶段为 false** —— text-to-speech LM 是单任务 next-token prediction 一种 CE loss [GitHub: cosyvoice/llm/llm.py:351-405 Qwen2LM.forward]"
training_data: "TTS 主训练 100 万 h，9 语种 + 19 中国方言 [§4.2]；tokenizer 训练 530K h，含 ASR(365K) / LID(85K) / SER(48K) / AED(21K) / SA(11K) [§2.1 + Tab.3]；指令跟随 SFT 数据 5,000 h，100+ 风格 [§2.5 + Tab.1]"
post_training: "**论文宣称**：DiffRO (Differentiable Reward Optimization) —— Token2Text reward + Gumbel-Softmax 可微 sample + token-level KL 约束；MTR 扩展到 SER/MOS/AED 多任务 reward [§2.2, Eq.3-7]. **开源 repo 实际公开**：仅有 DPO 实现（DPOLoss + forward_dpo），未找到 DiffRO 训练代码（grep 无 Gumbel-Softmax / Token2Text / forward_diff） [GitHub: cosyvoice/utils/losses.py:24-57; cosyvoice/llm/llm.py:407-456 forward_dpo]. **复现 DiffRO 收益需自行实现**"
codec_detail: "**FSQ-based codec, NOT RVQ**；codebook size 6561 = (2K+1)^D = 3^8 [§2.1 Eq.2; conf:26 verify K=1, D=8]；token frame rate 25 Hz [§2.1 末段; conf:13 token_frame_rate=25]；audio sample rate 24 kHz [conf:8]；Mel: hop=480, num_mels=80, n_fft=1920, win=1920 [conf:104-110]；token_mel_ratio=2 → mel frame rate 50 Hz [conf:14]"

# === 知识地图联动 ===
domain: TTS
subdomain: codec-lm-tts
routes: [codec-lm-tts, controllable-tts, instruction-tts, streaming-tts, voice-cloning]
problems: [zero-shot-cloning, prosody-control, multilinguality, data-scale, codec-design, instruction-following, latency, evaluation]
representations: [acoustic-token, semantic-token, mixed-token]
related_maps:
  - "[[TTS-技术路线图]]"
  - "[[TTS-表示层地图]]"
  - "[[TTS-代表模型谱系]]"
  - "[[TTS-核心挑战]]"
  - "[[TTS-评测体系]]"
  - "[[TTS-趋势判断]]"
related_surveys:
  - "[[ControllableTTS-Survey]]"
evidence_level: medium
maturity: emerging
last_repositioned: 2026-05-26

# === 回流状态 ===
map_backfilled: false
backfilled_at:

# === 资源本地化路径 ===
pdf_local: "~/DailyPaper/.cache/papers/2505.17589/paper.pdf"
html_local: "~/DailyPaper/.cache/papers/2505.17589/paper.html"
figures_dir: "_resources/2505.17589/figures/"
github_local: "~/DailyPaper/.cache/papers/2505.17589/github/FunAudioLLM_CosyVoice/"
cached_at: 2026-05-26

# === 通用元数据 ===
image_source: online
arxiv_html: https://arxiv.org/html/2505.17589v2
created: 2026-05-25
last_rewritten: 2026-05-26
---

# 论文笔记：CosyVoice 3: Towards In-the-wild Speech Generation via Scaling-up and Post-training

> **本笔记按工业 tech report 标准撰写**：重点抽取数据规模、训练 recipe、超参、serving 配置、可复现细节。工程价值 > 学术新颖性。

## 元信息

| 项目 | 内容 |
|------|------|
| 机构 | Speech Team, Tongyi Lab, Alibaba Group |
| 发布日期 | 2025-05 |
| 系列上下文 | [[CosyVoice]] (2024-07) → [[CosyVoice 2]] (2024-12) → **CosyVoice 3** (2025-05) |
| 项目主页 | [funaudiollm.github.io/cosyvoice3](https://funaudiollm.github.io/cosyvoice3) |
| 是否开源 | **代码开源**（GitHub FunAudioLLM/CosyVoice），**checkpoint 部分开源**（tokenizer ONNX + LM/CFM 模型）；DiffRO 训练代码未公开；1M h 训练数据闭源 |
| 链接 | [arXiv](https://arxiv.org/abs/2505.17589) / [Code](https://github.com/FunAudioLLM/CosyVoice) |
| 对比基线 | NAR: [[MaskGCT]] / E2 TTS / [[F5-TTS]] / F5R-TTS；AR: [[Seed-TTS]] / [[FireRedTTS]] / [[Qwen2.5-Omni]] / [[CosyVoice]] / [[CosyVoice 2]] / [[Spark-TTS]] |

---

## 一句话总结

> 阿里通义 CosyVoice 系列第三代：把训练数据从 17 万 h 扩到 100 万 h（9 语 + 19 方言），LM 从 0.5B 扩到 1.5B 并基于 Qwen2 warm-start，CFM 升 DiT 300M，加上多任务监督 FSQ tokenizer 和 DiffRO 后训练，把 in-the-wild zero-shot TTS 推到当前工业天花板（test-zh CER 0.71% / test-en WER 1.45%）。

---

## 核心工程交付（按工业价值排序）

| # | 交付 | 关键指标 | 出处 |
|---|---|---|---|
| 1 | **百万级多语种 TTS 训练数据 + 6 步管线** | 1M h, 9 lang + 19 dialect | §3 + §4.2 |
| 2 | **SEED-TTS-eval 新 SOTA** | test-zh CER **0.71%**（v2: 1.45%，-51%）；test-en WER **1.45%**（v2: 2.57%，-44%） | Tab.4 |
| 3 | **DiffRO 后训练范式** | 跨数据集 12-35% 相对 WER 提升；Korean cross-lingual -68.6% | §2.2 + Tab.4-7 |
| 4 | **MinMo-based 多任务 FSQ tokenizer** | 25 Hz / 6561 vocab / 单 codebook；下游 TTS 在 3K h 数据上即超 v2 在同等数据上的成绩 | §2.1 + Tab.12 |
| 5 | **CV3-Eval 多语言公开 benchmark** | 9 lang × 500 样本 + 跨语言 + 情感 + 方言 + 困难子集 | §4.4 |
| 6 | **Pronunciation Inpainting**（BPE TTS 的可控发音方案） | 中英多音字 100% 纠正率 | Tab.13 |
| 7 | **流式 chunk-aware CausalCFM** | chunk=25 token / streaming 推理可行 | conf:17, 38-75 |

---

## 1. 系统架构总览

[已 verify §2 + GitHub conf cosyvoice3.yaml]

```
Text → Qwen2 Tokenizer → text tokens
                              ↓
        Qwen2-warm-start Text-to-Speech LM (0.5B / 1.5B)
                              ↓
                    25 Hz Speech Tokens (vocab 6561)
                              ↓
          Chunk-aware Causal CFM (DiT 300M backbone)
                              ↓
                    50 Hz Mel-Spectrogram (80-dim, 24 kHz)
                              ↓
              Causal HiFi-GAN Vocoder (with NSF F0 predictor)
                              ↓
                          24 kHz Waveform
```

**与 v2 的关键变化**：
- ❌ 移除 v2 的独立 text encoder + length regularization
- ❌ tokenizer 基座从 SenseVoice-Large 换成 **MinMo**
- ✓ LM 从 0.5B 扩到 1.5B
- ✓ CFM 从 100M U-Net 升 **DiT 300M**
- ✓ token / mel 帧率不匹配用简单 interpolation 解决（而非 length regulator）
- ✓ 新增 DiffRO 后训练
- ✓ 数据从 ~170K h 扩到 **1M h**

---

## 2. 监督多任务 Speech Tokenizer

### 2.1 设计变化（v2 → v3）

[已 verify §2.1 paper.html L286-289]

| 维度 | CosyVoice 2 | **CosyVoice 3** |
|---|---|---|
| 基座 | SenseVoice-Large（纯 ASR） | **MinMo**（1.4M h 多任务 speech understanding） |
| 监督任务 | ASR-only | **5 任务**：ASR / LID / SER / AED / SA |
| Tokenizer 数据 | （未明示） | **530K h** |
| 量化方式 | FSQ | FSQ |
| 帧率 | 25 Hz | **25 Hz** |
| 码本大小 | 6561 | **6561** = 3^8 (K=1, D=8) |

### 2.2 架构与训练流程

[已 verify §2.1 paper.html L290-310]

```
input speech X
    ↓
Voice Encoder₁  (12-layer Transformer + RoPE)
    ↓ H (intermediate representation)
[FSQ Module]    Proj_down → ROUND → Proj_up
    ↓ Ĥ (quantized)                  [训练用 STE 近似梯度]
Voice Encoder₂  (MinMo 余下模块)
    ↓
MinMo LLM       预测文本 token 后验
```

### 2.3 关键公式

**FSQ 量化**（Eq.1）：

$$
\bar{H} = \mathrm{ROUND}(\mathrm{Proj}_{down}(H))
$$

$$
\hat{H} = \mathrm{Proj}_{up}(\bar{H})
$$

**Speech Token 索引**（Eq.2）：

$$
\mu_i = \sum_{j=0}^{D-1} \bar{h}_{i,j}\,(2K+1)^j
$$

→ 把 D 维量化向量编码为 $(2K+1)$ 进制的单 codebook 标量索引。**$Q = (2K+1)^D = 6561$**

### 2.4 五任务训练数据（Table 3）

| 任务 | 时长 |
|---|---|
| 多语种 ASR（zh/en/ja/ko/ru/fr/de） | 365K h |
| LID 语言识别 | 85K h |
| SER 语音情感识别 | 48K h |
| AED 音频事件检测 | 21K h |
| SA 说话人分析 | 11K h |
| **合计** | **530K h** |

### 2.5 Tokenizer Ablation（Table 10-12）

#### Table 10: 上游 ASR 任务上 tokenizer 退化对比

| 方法 | C.V. EN | C.V. CN | C.V. JA | C.V. KO | Fleurs EN | Fleurs CN |
|---|---|---|---|---|---|---|
| [[SenseVoice]] | 7.70 | 8.67 | – | – | 4.57 | 6.98 |
| [[MinMo]] | 7.36 | 8.56 | – | – | 4.43 | 6.71 |
| VQ-SenseVoice | 18.26 | 11.56 | – | – | 7.65 | 5.03 |
| FSQ-SenseVoice | 10.67 | 7.29 | – | – | 6.58 | 4.43 |
| **FSQ-MinMo (CV3)** | 11.36 | 9.21 | 13.90 | 9.78 | 4.46 | **3.35** |

**反直觉发现**：FSQ-MinMo 在 Fleurs CN 上的 WER (3.35) **甚至低于**未量化的 MinMo (6.71)。FSQ 的信息瓶颈对特定任务起到正则化效果。

#### Table 11: 副语言任务（AIR-Bench）

| 方法 | LID | Gender | Age | Emotion | Vocal Sound | Sound QA |
|---|---|---|---|---|---|---|
| [[MinMo]] | 99.2 | 84.8 | 70.1 | 62.4 | 90.7 | 59.1 |
| FSQ-MinMo | **99.2** | 72.8 | 41.8 | **68.4** | 61.3 | 57.7 |

**关键观察**：FSQ 量化后 LID 几乎无损，**情感识别反而提升**（62.4 → 68.4），但性别 / 年龄 / vocal sound 显著退化。FSQ 选择性保留语义+情感信息，丢弃部分细粒度声学。

#### Table 12: 下游 TTS 性能（替换 token，固定 LM/CFM 架构）

| Model | test-zh CER↓ | test-zh SS↑ | test-en WER↓ | test-en SS↑ | test-hard CER↓ | test-hard SS↑ |
|---|---|---|---|---|---|---|
| **3000h 数据** | | | | | | |
| SoundStream (1st VQ) | 14.19 | 0.457 | 25.34 | 0.301 | 27.05 | 0.455 |
| [[HuBERT]] | 18.68 | 0.716 | 6.50 | 0.609 | 33.83 | 0.699 |
| w2v-BERT 2.0 | 2.62 | 0.381 | 6.72 | 0.261 | 23.89 | 0.374 |
| CosyVoice 2 | 1.92 | 0.668 | 7.21 | 0.535 | 15.99 | 0.645 |
| **CosyVoice 3-0.5B** | **1.68** | **0.710** | **6.60** | **0.614** | 27.60 | **0.679** |
| **170Kh 数据** | | | | | | |
| CosyVoice 2 | 1.45 | 0.806 | 2.57 | 0.736 | 6.83 | 0.776 |
| **CosyVoice 3-0.5B** | **1.27** | **0.815** | **2.46** | **0.747** | 6.96 | **0.787** |

**核心工程结论**：
- 声学 token (SoundStream) 缺乏语义信息 → 内容一致性差
- HuBERT 语义 token 有 SS 但中文 CER 极高（语言特异性）
- **监督多任务 tokenizer 在 CER + SS 两个维度同时领先**
- 3K → 170K h scaling 带来 **63-75% 相对 WER 提升**
- 1M h 进一步 scaling 收益开始 plateau

---

## 3. DiffRO 后训练（核心创新 + 复现警示）

### 3.1 设计动机

[已 verify §2.2 paper.html L320-330]

现有 TTS RL 方法（如 F5R-TTS 用 GRPO）的两个问题：
1. **计算开销大**：必须前向走完整个 CFM + Vocoder 计算链才能得到 reward
2. **奖励信号区分度低**：合成语音之间相似度高，positive/negative 难区分

DiffRO 的解法：**直接在 speech token 空间计算 reward，绕过 CFM + Vocoder**。

### 3.2 三个关键技术

1. **Token2Text reward model**：训一个 ASR-like 模型，把 speech token 序列映射到文本，用后验概率做 reward
2. **Gumbel-Softmax 离散采样**：使 LLM 输出的离散 token 可微，能直接反向传播（不走 RL training loop）
3. **Token-level KL（不是 sequence-level）**：在每个时间步 logits 上算 KL，计算量更小更稳定

### 3.3 关键公式

**Gumbel-Softmax 采样**（Eq.3）：

$$
\tilde{\mu}_t = \mathrm{GumbelSoftmax}\, P_{\pi_\theta}(\mu_t \mid \mu_{1:t-1}; Y)
$$

**ASR reward**（Eq.4）：

$$
R_{\mathrm{ASR}}(Y) = \log P_{\mathrm{ASR}}(\tilde{Y}_n = Y_n \mid Y_{1:n-1}; \tilde{\mu}_{1:T})
$$

**优化目标**（Eq.5）：

$$
\pi_\theta^* = \max_{\pi_\theta} \mathbb{E}[R(Y)] - \beta\, D_{\mathrm{KL}}[\pi_\theta(\mu \mid Y) \,\|\, \pi_{\mathrm{ref}}(\mu \mid Y)]
$$

**Token 级 KL**（Eq.6）：

$$
D_{\mathrm{KL}}[\pi_\theta(\mu \mid Y) \,\|\, \pi_{\mathrm{ref}}(\mu \mid Y)] = \sum_{t=1}^{T} \sum_{k=0}^{Q} P_{\pi_\theta}(\mu_t=k) \log \frac{P_{\pi_\theta}(\mu_t=k)}{P_{\pi_{\mathrm{ref}}}(\mu_t=k)}
$$

**MTR (Multi-Task Reward)**（Eq.7）：

$$
R_{\mathrm{MTR}}(Y, \{A_i\}_{i=1}^{K}) = \sum_i \log P_{\mathrm{task}_i}(\tilde{A_i} = A_i \mid \tilde{\mu})
$$

→ 除 ASR 外，还可加入 SER / MOS prediction / AED 等多任务 reward 信号。**DiffRO-EMO 就是 SER reward 实例化**。

### 3.4 ⚠️ 关键复现警示

[已 verify GitHub: cosyvoice/utils/losses.py + cosyvoice/llm/llm.py 完整文件]

| 论文宣称 | 开源 repo 实际 |
|---|---|
| DiffRO 公式 + 实验 [§2.2 + Tab.4-9] | ❌ 未公开训练代码 |
| Token2Text model | ❌ grep 无相关实现 |
| Gumbel-Softmax 采样 | ❌ grep 无相关代码 |
| Token-level KL | ❌ 未实现 |
| MTR 多任务 reward 加权 | ❌ 未实现 |
| DPO baseline | ✅ DPOLoss + forward_dpo（`cosyvoice/utils/losses.py:24-57` + `cosyvoice/llm/llm.py:407-456`） |

**结论**：复现表 4-9 中 `+ DiffRO` / `+ RL` 的提升需**自行实现** Token2Text + Gumbel-Softmax 训练 loop。论文 §2.2 是公开的 spec，但没有可即取即用的 reference 实现。

---

## 4. Pronunciation Inpainting（工程级发音可控性）

### 4.1 问题

LLM-based TTS 用 BPE tokenizer，对多音字 / 罕见词缺乏发音控制（vs phoneme-based 方法）。

### 4.2 方案

[已 verify §2.3]

扩展 tokenizer 词表支持 **词 + 音素混合序列**。三种构造方式对比（Tab.13）：

| 方法 | zh 错误数 | zh 纠正数 | zh 纠正率(%) | en 错误数 | en 纠正数 | en 纠正率(%) |
|---|---|---|---|---|---|---|
| RepAll + MixPhn | 13 | 9 | 69.2 | 11 | 8 | 72.7 |
| **RepMono + MixPhn** | **15** | **15** | **100** | **9** | **9** | **100** |
| RepMono + CatPhn | 15 | 13 | 86.7 | 8 | 8 | 100 |

- **RepAll**：所有字符随机替换为 G2P 音素，存在 G2P 模型误差
- **RepMono**：**只替换单音字 / 单音词**，确保训练标签准确
- **MixPhn**：用音素直接替换字符
- **CatPhn**：保留字符 + concat 音素

→ **RepMono + MixPhn 中英文均达 100% 纠正率**

---

## 5. 多语种数据管线（六步，可工程化复刻）

[已 verify §3] 100 万 h in-the-wild 数据，来自 Internet 有声书 / 视频 / 播客。

| Step | 操作 | 工具 / 阈值 |
|---|---|---|
| **1. Speech detection + segmentation** | 说话人日志 + VAD + 音频事件检测 → speaker-level segment ≤30s | 内部模块（可换开源等价） |
| **2. Noise reduction** | 降噪 + 基于首尾帧 energy 检测截断异常 → trim 首尾静音 | **MossFormer2** |
| **3. ASR transcription** | LID → 三 ASR 系统转录 → 交叉验证 | **Faster-Whisper Large-V3**（LID）+ Whisper Large-V3 / **NVIDIA NeMo Canary-1B** / **Meta seamlessM4T-V2-large**；保留 avg pairwise WER < 15% |
| **4. Punctuation adjustment** | 基于词间停顿调整标点 | **MFA**：≥300ms 加逗号；≤50ms 删除停顿标点（逗号/分号/冒号/句号/问号/叹号） |
| **5. Volume standardization** | 简单峰值归一化 | $\mathrm{norm\_wav} = \frac{\mathrm{raw\_wav}}{\max(\mathrm{raw\_wav})} \times 0.6$（Eq.8） |
| **6. Length-ratio filter** | 计算 speech token 长度 / text token 长度比 → 丢弃最小 1% 和最大 5% | 排除"短音频长文本" / "长音频短语音"异常 case |

> 工程意义：这是**端到端复刻 in-the-wild TTS 数据**的可执行 cookbook，每步工具都点名。

---

## 6. 训练超参（可复现 ground truth）

[已 verify GitHub: examples/libritts/cosyvoice3/conf/cosyvoice3.yaml]

### 6.1 全局参数

| 参数 | 值 |
|---|---|
| Sample rate | 24 kHz |
| Mel hop size | 480 (→ 50 Hz mel frame rate) |
| Mel n_fft | 1920 |
| Mel num_mels | 80 |
| Speech token vocab | 6561 (FSQ output) |
| Speech token frame rate | 25 Hz |
| token_mel_ratio | 2 |
| Streaming chunk_size | 25 tokens |
| num_decoding_left_chunks | -1 (use all left) |
| Random seed | 1986 |

### 6.2 LM 配置

| 参数 | 值 |
|---|---|
| Base class | `CosyVoice3LM(Qwen2LM)` |
| LM 内核 | `Qwen2Encoder` → `Qwen2ForCausalLM.from_pretrained(qwen_pretrain_path)` |
| llm_input_size / output_size | 896 / 896 |
| Speech embed vocab | 6561 + 200 = 6761（含 sos/eos/task_id/fill + 余量） |
| Loss | `LabelSmoothingLoss`（lsm_weight=0, length-normalized） |
| Mix ratio | [5, 15] |
| Sampling | `ras_sampling` top_p=0.8, top_k=25, win_size=10, tau_r=0.1 |
| Optimizer | Adam |
| LR | 1e-5（SFT 同） |
| Scheduler | constantlr，warmup_steps=2500 |
| Max epoch | 200 |
| Grad clip / accum_grad | 5 / 2 |

### 6.3 CFM 配置

| 参数 | 值 |
|---|---|
| Class | `CausalMaskedDiffWithDiT` |
| Backbone | DiT (`cosyvoice/flow/DiT/dit.py`) |
| dim / depth / heads / dim_head | 1024 / 22 / 16 / 64 |
| ff_mult | 2 |
| Solver | euler |
| t_scheduler | cosine |
| training_cfg_rate | 0.2 |
| inference_cfg_rate | 0.7 |
| reg_loss_type | l1 |
| sigma_min | 1e-6 |
| only_mask_loss | True |
| pre_lookahead_len | 3 |
| spk_embed_dim | 192 |

### 6.4 Vocoder（HiFi-GAN with NSF）

| 参数 | 值 |
|---|---|
| Class | `CausalHiFTGenerator` |
| Base channels | 512 |
| nb_harmonics | 8 |
| Upsample rates | [8, 5, 3] |
| Upsample kernel sizes | [16, 11, 7] |
| iSTFT (n_fft / hop) | 16 / 4 |
| Resblock kernel | [3, 7, 11] |
| F0 predictor | `CausalConvRNNF0Predictor`（in=80, cond=512） |
| Discriminator | MPD (Multi-Period) + MRD (Multi-Resolution Spec) |
| GAN LR (gen + disc) | 2e-4（accum_grad 必须 = 1） |

---

## 7. 模型缩放：v3-0.5B vs v3-1.5B

[已 verify §4.2]

| 维度 | 0.5B | 1.5B |
|---|---|---|
| LM 参数 | 0.5B (Qwen2-0.5B warm-start) | **1.5B** |
| CFM 参数 | 100M → 300M | **300M (DiT)** |
| Tokenizer | 同（MinMo + FSQ） | 同 |
| 训练数据 | 1M h | 1M h |
| 后训练 | DiffRO 同 | DiffRO 同 |

→ 论文承认困难场景下 1.5B 反而不如 0.5B（CV3-Eval hard-zh: 14.15 vs 9.77），归因于"困难样本数据不足"，计划扩到 tens of millions of hours。

---

## 8. 实验结果（按 benchmark 维度）

### 8.1 SEED-TTS-eval（Table 4，主战场）

| Model | test-zh CER↓ | test-zh SS↑ | test-en WER↓ | test-en SS↑ | test-hard CER↓ | test-hard SS↑ |
|---|---|---|---|---|---|---|
| Human | 1.26 | 0.755 | 2.14 | 0.734 | – | – |
| Vocoder Resyn | 1.27 | 0.720 | 2.17 | 0.700 | – | – |
| MaskGCT | 2.27 | 0.774 | 2.62 | 0.714 | 10.27 | 0.748 |
| E2 TTS (32 NFE) | 1.97 | 0.730 | 2.19 | 0.710 | – | – |
| F5-TTS (32 NFE) | 1.56 | 0.741 | 1.83 | 0.647 | 8.67 | 0.713 |
| F5R-TTS | 1.37 | 0.754 | – | – | 8.79 | 0.718 |
| Seed-TTS | 1.12 | **0.796** | 2.25 | **0.762** | 7.59 | **0.776** |
| FireRedTTS | 1.51 | 0.635 | 3.82 | 0.460 | 17.45 | 0.621 |
| Qwen2.5-Omni-7B | 1.70 | 0.752 | 2.72 | 0.632 | 7.97 | 0.747 |
| Qwen2.5-Omni-7B_RL | 1.42 | 0.754 | 2.33 | 0.641 | 6.54 | 0.752 |
| CosyVoice | 3.63 | 0.723 | 4.29 | 0.609 | 11.75 | 0.709 |
| CosyVoice 2 | 1.45 | 0.748 | 2.57 | 0.652 | 6.83 | 0.724 |
| Spark-TTS | 1.20 | 0.672 | 1.98 | 0.584 | – | – |
| **CV3-0.5B** | 1.16 | 0.780 | 2.02 | 0.718 | 6.08 | 0.758 |
| **CV3-0.5B_RL** | **0.75** | 0.774 | 1.76 | 0.695 | **5.09** | 0.750 |
| **CV3-1.5B** | 1.12 | 0.781 | 2.21 | 0.720 | 5.83 | 0.758 |
| **CV3-1.5B_RL** | **0.71** | 0.775 | **1.45** | 0.695 | 5.66 | 0.750 |

**关键工程指标**：
- v2→v3 相对提升：**test-zh CER -51%**（1.45 → 0.71）/ **test-en WER -44%**（2.57 → 1.45）/ test-hard -26%（6.83 → 5.09）
- **唯一超 Human（test-zh CER 1.26）的系统**（v3-1.5B_RL 0.71）
- DiffRO 后训练贡献 **12-35% 相对提升**
- ⚠️ **SS 上 Seed-TTS 仍领先**（v3 缩小差距但未超），作者归因于 speaker diversity + pretrain 数据规模差距
- ⚠️ RL 会**轻微降低 SS**（reward hacking：0.780 → 0.774），未解决

### 8.2 CV3-Eval Multilingual Voice Cloning（Table 5）

| Model | zh | en | ja | ko | de | es | fr | it | ru |
|---|---|---|---|---|---|---|---|---|---|
| F5-TTS | 5.47 | 8.90 | – | – | – | – | – | – | – |
| Spark-TTS | 5.15 | 11.0 | – | – | – | – | – | – | – |
| GPT-SoVITS | 7.34 | 12.5 | – | – | – | – | – | – | – |
| CV2 | 4.08 | 6.32 | 9.13 | 19.7 | – | – | – | – | – |
| + DiffRO | 3.00 | 4.72 | 6.36 | 5.14 | – | – | – | – | – |
| **CV3-0.5B** | 3.89 | 5.24 | 10.4 | 12.8 | 7.41 | 4.25 | 12.9 | 6.68 | 6.77 |
| + DiffRO | **2.89** | **3.68** | **5.15** | **4.02** | **4.51** | **2.99** | **8.56** | **2.94** | **3.79** |
| CV3-1.5B | 3.91 | 4.99 | 7.57 | 5.69 | 6.43 | 4.47 | 11.8 | 10.5 | 6.64 |
| + DiffRO | 3.01 | 3.71 | 5.27 | 4.01 | 3.93 | 3.26 | 8.09 | 2.72 | 4.11 |

**关键观察**：
- **CV3 是唯一覆盖全部 9 语种的系统**
- DiffRO 在所有语种全部正向提升，**Korean 提升最大**：CV3-0.5B 12.8 → 4.02（**-68.6%**）
- CV2 在 Korean 上 19.7% WER 是因为韩语训练数据不足

### 8.3 CV3-Eval 困难样本（Table 6）

| Model | hard-zh WER↓ | hard-zh SS↑ | hard-zh DNSMOS↑ | hard-en WER↓ | hard-en SS↑ | hard-en DNSMOS↑ |
|---|---|---|---|---|---|---|
| CV2 | 12.58 | 72.6 | 3.81 | 11.96 | 66.7 | 3.95 |
| + DiffRO | 10.66 | 71.7 | 3.81 | 10.25 | 62.4 | 3.97 |
| CV3-0.5B | 14.15 | 78.6 | 3.75 | 9.04 | 75.9 | 3.92 |
| + DiffRO | **8.26** | 77.8 | 3.80 | **7.60** | 73.9 | 3.95 |
| CV3-1.5B | 9.77 | 78.5 | 3.79 | 10.55 | 76.1 | 3.95 |
| + DiffRO | 9.06 | **78.2** | **3.81** | 7.56 | **74.6** | 3.95 |

> 困难样本（罕见词 / 绕口令 / 领域术语）上 DiffRO 提升明显小于普通样本，作者明确点出"hard sample 对 reward model 是挑战"。

### 8.4 跨语言克隆（Table 7-8）

**Table 7** 12 个跨语言组合 WER：CV3-1.5B + DiffRO **全部 12 个条件最优**。半数条件下 DiffRO 带来 >50% 相对提升。

**Table 8** zh↔en 跨语言主流对比：

| Model | en→zh WER↓ | en→zh SS↑ | en→zh MOS↑ | zh→en WER↓ | zh→en SS↑ | zh→en MOS↑ |
|---|---|---|---|---|---|---|
| F5-TTS | 11.6 | 64.2 | 3.77 | 5.57 | 64.7 | 3.77 |
| Spark-TTS | 12.4 | 48.4 | 3.65 | 7.36 | 56.7 | 3.61 |
| CosyVoice 2 | 13.5 | 63.3 | 3.87 | 6.47 | 64.3 | 3.75 |
| CV3-0.5B | 8.48 | 67.4 | 3.82 | 4.99 | 67.8 | 3.75 |
| **CV3-1.5B** | **8.01** | **66.9** | **3.83** | **4.32** | **66.4** | **3.77** |

### 8.5 情感克隆（Table 9）

| Model | TR-happy | TR-sad | TR-angry | TF-happy | TF-sad | TF-angry |
|---|---|---|---|---|---|---|
| F5-TTS | 0.92 | 0.52 | 0.72 | 0.80 | 0.28 | 0.64 |
| Spark-TTS | 0.80 | 0.56 | 0.50 | 0.50 | 0.60 | 0.36 |
| GPT-SoVITS | 0.88 | 0.54 | 0.50 | 0.48 | 0.40 | 0.30 |
| CosyVoice 2 | 0.84 | 0.72 | 0.58 | 0.56 | 0.44 | 0.38 |
| CV3-0.5B | 0.92 | 0.70 | 0.72 | 0.64 | 0.42 | 0.58 |
| CV3-1.5B | 0.86 | 0.64 | 0.72 | 0.64 | 0.44 | 0.48 |
| **+ DiffRO-EMO** | **0.98** | 0.68 | **0.84** | **0.98** | 0.50 | **0.68** |

**关键发现**：
- "Text-Unrelated 情感"远难于"Text-Related" → TTS 主要从**文本语义**推断情感
- DiffRO-EMO（SER 任务 reward）大幅提升 happy / angry，但 sad 仍是挑战

### 8.6 指令式语音生成（Table 14）

| Model | Expresso WER↓ | Expresso SIM↑ | Expresso MOS↑ | Internal WER↓ | Internal SIM↑ | Internal MOS↑ |
|---|---|---|---|---|---|---|
| GroundTruth | 10.0 | 100 | 3.65 | 8.98 | 100 | 3.47 |
| CosyVoice 2 | 9.42 | 60.98 | 3.54 | 7.75 | 72.99 | 3.53 |
| CV3-0.5B | 13.72 | 67.82 | 3.56 | 7.30 | 80.45 | 3.51 |
| **CV3-1.5B** | 13.43 | **68.25** | 3.56 | **7.31** | **81.06** | 3.51 |

**关键**：style similarity 相对 v2 提升约 **11%**（72.99 → 81.06）。Expresso WER 偏高，作者归因于 ASR 模型偏好标准发音（表达性语音含更多非标准发音）。**歌唱尚未支持**。

---

## 9. 结果可信度分层

| 可信度 | 结果 | 理由 |
|---|---|---|
| **高** | SEED-TTS-eval 全部指标 | 公开 benchmark + 10 baseline + 公开 SS evaluator（WavLM/ERes2Net）+ 代码可复现（除 DiffRO 训练 loop） |
| **高** | Tokenizer ablation (Tab.10-12) | 公开 benchmark + 替换 token 控制变量 |
| **中** | CV3-Eval 多语 / 跨语 / 困难 | benchmark 已发布（本身就是贡献），但 baseline 多数不支持小语种导致 CV3 没有公平对比 |
| **中** | 情感克隆（Table 9） | emo2vec-large-plus 作分类器，是开源工具但结果对分类器敏感 |
| **中-低** | DiffRO 提升数字（+ DiffRO 列） | 论文公式完整，但**实现代码未在 repo 公开**，第三方需自行复刻 |
| **低** | 100 万 h 训练数据的质量 | 数据完全闭源 + 6 步管线只给原理无具体阈值（VAD 时长 / MossFormer2 阈值等部分缺） |
| **低** | 流式 RTF / 首包延迟 | **论文未报告任何延迟数字** |

---

## 10. 批判性思考（工程视角）

### 10.1 核心 Claim 审查

1. **Paper Claim**: "We propose a novel speech tokenizer derived from a large audio understanding LLM"
   **My Assessment**: 把 SenseVoice 换成 MinMo 是路径延续而非创新，但**信息瓶颈使情感识别反而提升**（Tab.11 62.4 → 68.4）这一发现有价值。FSQ + 多任务监督的组合在工业上是经过验证的路线（v2 已用 FSQ）。

2. **Paper Claim**: "DiffRO ... applicable not only to the CosyVoice series but also to other discrete-token-based speech synthesis models"
   **My Assessment**: 公式上确实通用（Token2Text + Gumbel-Softmax + token-level KL），**但开源 repo 未提供训练代码**——这与 claim 的"applicable"在工程实践上有 gap。论文有 spec，但没有可即取即用的 baseline。

3. **Paper Claim**: "achieves state-of-the-art results on multiple benchmarks"
   **My Assessment**: SEED-TTS-eval 上 CER/WER 确实 SOTA，**且超 Human (test-zh CER 0.71 < 1.26)**。但 SS 仍落后 Seed-TTS，作者承认。CV3-Eval 多语种因 baseline 不支持小语种导致"独此一家"——是 first-mover 优势而非真正横向碾压。

### 10.2 工程优点

1. **数据 + 模型双 scaling 实证**：1M h 实测 + LM 0.5B → 1.5B 可见效果，是 TTS 领域 scaling law 难得的工业级实证。
2. **Tokenizer ablation 极扎实**：3K h / 170K h 两规模 × 5 种 tokenizer 完整对比（Tab.12），是 codec 设计研究的优秀参考。
3. **数据管线可复刻**：6 步流程 + 每步点名工具（MossFormer2 / Faster-Whisper / Canary / seamlessM4T / MFA）→ 第三方可以照搬。
4. **DiffRO 思路精巧**：绕过 CFM/Vocoder 算 reward 是真正的工程级 insight，理论上可迁移到任何 LLM-based TTS。
5. **CV3-Eval 公开**：包含 9 语 / 跨语 / 情感 / 方言 / 困难子集，弥补 SEED-TTS-eval 仅覆盖 zh/en 的缺口。
6. **诚实暴露问题**：明确报告 RL reward hacking (SS ↓) / Expresso WER 偏高的 ASR bias / 1.5B 困难样本不如 0.5B / 歌唱未支持 → 这是工业 tech report 该有的态度。

### 10.3 工程局限

1. **DiffRO 代码不开源**：是最大复现障碍。论文 Table 4-9 中 `+ DiffRO` / `+ RL` 列的提升对外部团队是 "看到但摸不到"。
2. **Tokenizer 是 ONNX 黑盒**：`speech_tokenizer_v3.batch.onnx` 不公开训练代码 + FSQ 投影维度等内部超参未披露 → 想做自定义 token vocab 或换基座完全做不了。
3. **1M h 训练数据闭源**：管线步骤清楚但具体数据源 / 清洗阈值 / 过滤比例不可见。
4. **音色（timbre）无法通过文本指令控制**：Limitations 明确承认。这是 role-playing 场景的硬伤。
5. **歌唱生成不佳**：tokenizer 和 LM 训练均未针对歌唱优化。
6. **流式延迟未报告**：作为 v2 的续作（v2 主打 streaming），v3 没给任何首包延迟 / RTF 数字。**1.5B 模型 streaming 是否仍可用是个问号**。
7. **RL reward hacking 未解决**：DiffRO 提升 WER 但轻微降低 SS，论文承认且未给最终方案。

### 10.4 可复现性评估

- [x] **代码开源**（cosyvoice/ 主模块 + cosyvoice3 conf）
- [x] **LM/CFM checkpoint 部分开源**（FunAudioLLM 仓库提供 hub 下载）
- [-] **Tokenizer 仅 ONNX 推理**（无训练代码 / 内部架构）
- [-] **DiffRO 实现不开源**
- [ ] **1M h 训练数据不开源**
- [x] **CV3-Eval 测评集公开**（贡献本身）
- [x] **架构描述完整**（含具体超参，conf 文件公开）
- [-] **后训练数据 SFT 5000h instruction data 不开源**

---

## 11. 与同期工业 tech report 对照

| 维度 | CosyVoice 3 | StepAudio 2.5 | Qwen3-TTS |
|---|---|---|---|
| 定位 | 独立 TTS 产品 | Unified ASR/TTS/Realtime | 独立 TTS 产品 |
| LM init | Qwen2 warm-start（已 verify GitHub）| 文本 MoE LLM warm-start | Qwen3 warm-start（paper claim） |
| Codec | FSQ 单 codebook 6561 / 25 Hz | 未披露 | 双 codebook / 多 sub_talker |
| 主训练数据 | 1M h | 2.2T tokens (≈ 包含 800B speech) | 未明示 |
| 流式 | CausalCFM chunk=25 | (未明示具体首包延迟) | 双 codebook 同步友好 streaming |
| 评测 | SEED-TTS-eval + 自建 CV3-Eval | 仅 arena win-rate 67.6% | (各自 benchmark) |
| 后训练 | DiffRO（公式公开，代码未开源）| GRM-shaped RLHF | (各自方案) |
| 开源程度 | 代码 + 部分 ckpt + benchmark | **全闭源** | 部分开源 |
| TTS 客观指标 | 完整（CER/WER/SS/MOS）| **无客观指标，仅 arena** | (各自) |

→ 在工业 tech report 的"工程透明度"维度上，**CosyVoice 3 > Qwen3-TTS > StepAudio 2.5**。

---

## 🗺️ 在知识地图中的定位

- **所属领域**：[[TTS-领域总览]]（独立 TTS 系统，主要服务工业 voice cloning / configurable TTS API）
- **技术路线**：
  - [[TTS-技术路线图]] §路线 2 Codec LM + §路线融合 "Codec LM + Flow Matching"
  - [已 verify GitHub] LM 是 **Qwen2 warm-start**，与 v1 的 custom TransformerLM 完全不同；v1 → v2/v3 是 init 范式的关键跃迁
- **核心问题**：
  - [[TTS-核心挑战]] §挑战 1 零样本克隆 — Tab.4 工业代表
  - [[TTS-核心挑战]] §挑战 4 训练数据规模 — 100 万 h，9 语 + 19 方言
  - [[TTS-核心挑战]] §挑战 5 Codec 设计 — FSQ 6561 vocab 单 codebook（v2/v3 沿用同一码本设计）
  - [[TTS-核心挑战]] §挑战 3 流式延迟 — CausalCFM chunk=25 已实现，但延迟数字未给
  - [[TTS-核心挑战]] §挑战 2 韵律 / 自然度 — DiffRO 后训练改善内容一致性；MTR 支持情感 reward
- **表示层位置**：
  - [[TTS-表示层地图]] §FSQ — 工业代表
  - [[TTS-表示层地图]] §混合 token — 多任务监督让单 codebook 同时承载内容 + 副语言
  - [[TTS-表示层地图]] §LM scaling 友好性 — 6561 单 codebook + 25 Hz 帧率，自回归极友好
- **在 SpeechLM/对话框架内的位置**：[[TTS-SpeechLM-Dialogue关系]] **位置 ① 独立产品**（vs StepAudio 2.5 位置 ①②③ 全覆盖）
- **相邻工作**：[[CosyVoice 2]] / [[CosyVoice]] / [[Qwen3-TTS]] / [[Seed-TTS]] / [[F5-TTS]] / [[Spark-TTS]] / [[MaskGCT]] / [[StepAudio2.5]]（同期对照）

---

## 🔄 后续重估

- **2026-05-25**：初读，建立核心技术框架。
- **2026-05-26（dogfood §11 三层 verify 后修订）**：通过 GitHub 代码 verify 关键修正：
  - **(修正 1) LM 初始化**：**确认 warm-start from Qwen2**（不是 cold-start 自定义 LM）
  - **(修正 2) DiffRO 公开度**：**论文公式完整，但开源 repo 实际只有 DPO 实现**——这是核心复现障碍
  - **(修正 3) Tokenizer**：**ONNX 黑盒**，FSQ 投影维度等内部细节无 L2 verify
- **2026-05-26（从头重写为工业 tech report 格式）**：把工程细节（conf 文件具体超参、DiT 配置、Vocoder 配置、GAN 训练参数）补全；6 步数据管线点名工具；DiffRO 复现警示前置；与 StepAudio 2.5 / Qwen3-TTS 横向对照；强调"DiffRO 公式 vs 代码"的 gap 作为复现 blocker。整体定位：**工业开源 TTS 工程透明度的天花板代表**——架构 + 超参 + benchmark 都公开，唯一 gap 是 DiffRO 训练 loop 和 1M h 数据。

---

## 关联笔记

### 基于
- [[CosyVoice 2]]：直接前作，沿用 LLM + chunk-aware CFM + HiFi-GAN 三段式
- [[CosyVoice]]：系列初代，提出 supervised semantic token
- [[MinMo]]：speech tokenizer 的基座（1.4M h 多任务 speech understanding LLM）
- [[Qwen2]]：LM warm-start 基座（已 verify GitHub）

### 对比
- [[StepAudio2.5]]：同期工业 tech report 对照（unified 路线）
- [[Qwen3-TTS]]：同期阿里另一条产品线
- [[F5-TTS]]：NAR 纯 flow matching 代表
- [[MaskGCT]]：NAR masked generative codec transformer
- [[Seed-TTS]]：AR 闭源标杆（SS 仍领先）
- [[Spark-TTS]]：单 LLM + BiCodec 路线
- [[FireRedTTS]]：小红书工业 TTS

### 方法相关
- [[FSQ]]：Finite Scalar Quantization
- [[DiffRO]]：核心后训练方法（公式见本笔记 §3.3）
- [[Flow Matching]] / [[DiT]]：CFM decoder
- [[Gumbel-Softmax]]：使离散 token 采样可微
- [[Speech Tokenizer]]
- [[ERes2Net]] / [[WavLM]]：SS 评测器

### 数据/工具相关
- [[Seed-TTS-eval]] / [[CV3-Eval]]
- [[Common Voice]] / [[Fleurs]] / [[EmoBox]]
- [[MossFormer2]]（降噪）
- [[Faster-Whisper]] / [[NeMo Canary]] / [[seamlessM4T]]（ASR 交叉验证）
- [[MFA]]（强制对齐 + 标点调整）
- [[Qwen-Max]]（TN/ITN 数据合成）

---

## 速查卡片

> [!summary] CosyVoice 3 (Alibaba Tongyi, 2025-05)
> - **核心**：监督多任务 FSQ tokenizer + DiffRO 后训练 + 1M h scaling，实现 9 语种 in-the-wild zero-shot TTS SOTA
> - **架构**：Qwen2 warm-start LM (0.5B / 1.5B) → 25 Hz speech token (vocab 6561 FSQ) → DiT-based CausalCFM (300M) → CausalHiFiGAN
> - **数据**：tokenizer 530K h 5 任务 / LM 1M h 9 lang + 19 dialect / 指令 SFT 5K h 100+ 风格
> - **结果**：SEED-TTS-eval test-zh CER 0.71%（v2: 1.45%，**-51%**）/ test-en WER 1.45%（v2: 2.57%，**-44%**）/ test-hard 5.09%（-26%）/ Korean cross-lingual DiffRO -68.6%
> - **开源程度**：✅ 代码 + 部分 ckpt + CV3-Eval；❌ DiffRO 训练 loop / 1M h 数据 / tokenizer 训练代码
> - **复现警示**：论文 §2.2 DiffRO 实现**未在开源 repo 出现**，仅有 DPO；表 4-9 `+ DiffRO` 列提升需自行实现 Token2Text + Gumbel-Softmax
> - **限制**：无法通过文本控制音色 / 不支持歌唱 / RL 有 SS 下降的 reward hacking / 流式延迟未报告

---

*笔记创建时间: 2026-05-25*
*从头重写为工业 tech report 格式: 2026-05-26*
