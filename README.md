# DailyPaper

我自动追踪 **TTS / 全双工 / Omni / Audio LLM / ASR** 方向论文的个人笔记仓库。

每天早 9:00、每周五中午，本地 Mac 上的一套 Claude Code Skills 流水线会自动：
1. 从 HuggingFace Daily / arXiv 抓近 24 小时新论文
2. 按关键词打分排序，挑出 Top 15
3. 让 Claude 写毒舌锐评 + 分流（必读 / 值得看 / 可跳过）
4. 给「必读」论文逐篇生成结构化笔记（公式、图表、概念链接）
5. 自动建概念库、刷 MOC 索引页
6. commit + push 到这个仓库

周五还会再跑一次跨日综述。

> 灵感与底座来自 [huangkiki/dailypaper-skills](https://github.com/huangkiki/dailypaper-skills)（机器人/具身智能方向），本仓库的定制工作把领域、关键词、概念分类、术语库换成了音频/语音方向，并补了 `paper-compare` / `paper-highlights` / `daily-papers-weekly` 三个 sub-skill。

---

## 📁 目录结构

```
.
├── DailyPapers/                      # 每日推荐报告
│   ├── 2026-MM-DD-论文推荐.md         # 一天一份，含分流表 + 逐篇锐评
│   ├── Weekly/
│   │   └── YYYY-Www.md               # 周报：跨日综述（周五自动）
│   ├── Archive/YYYY-MM/              # >30 天的旧报告自动归档于此
│   └── .history.json                 # 跨天去重（不要手改）
│
└── 论文笔记/                          # 所有精读笔记 + MOC 索引页
    ├── 1-TTS与语音合成/
    ├── 2-ASR与语音识别/
    ├── 3-Audio-Codec与Tokenizer/
    ├── 4-Vocoder与声码器/
    ├── 5-Speech-LLM与AudioLM/
    ├── 6-全双工与对话/
    ├── 7-Omni与多模态/
    ├── 8-Diffusion与FlowMatching/
    ├── 9-语音SSL与表示/
    ├── 10-语音翻译与跨语言/
    ├── 11-韵律与情感/
    ├── 12-数据集与评测/
    ├── 13-训练方法与对齐/
    ├── 14-LLM基础/
    ├── 15-其他音频任务/
    │
    ├── _概念/{同上 15 分类}/           # 跨论文累积的概念笔记（VALL-E / RVQ / Flow Matching ...）
    ├── _创新亮点/{按主题分文件}.md      # 跨论文累积的方案亮点合集（Codec设计 / 流式与低延迟 ...）
    ├── _对比报告/YYYY-MM-DD-A-vs-B.md  # 论文对比报告
    └── _待整理/                       # 自动分类失败的兜底
```

---

## 🐕 常用入口

在本地 Mac 上的 `/Users/xiangshu/DailyPaper/` 打开 Claude Code 后，自然语言就能触发：

| 想做什么 | 这么说 |
|---|---|
| 看今天有什么论文 | `今日论文推荐` |
| 看过去 3 天/一周 | `过去3天论文推荐` / `过去一周论文推荐` |
| 精读一篇 | `读一下这篇论文 <arXiv URL 或本地 PDF>` |
| 快速看 | `快速看一下 <URL>` |
| 批判性分析 | `批判性分析这篇论文 <URL>` |
| 跨论文对比 | `对比 [[CosyVoice]] [[F5-TTS]]` |
| 抽方案亮点入库 | `抽取 [[VibeVoice]] 的亮点入库` |
| 本周综述 | `本周论文总结` |
| 手动刷 MOC 索引 | `更新索引` |

---

## ⏰ 自动化时间表

| 任务 | 频率 | launchd Label |
|---|---|---|
| 每日抓取 + 推荐 + 必读笔记 | 工作日 + 周末每天 09:00 | `com.xiangshu.dailypaper` |
| 老报告归档 + 本周综述 | 周五 12:07 | `com.xiangshu.dailypaper.weekly` |

跑完会发桌面通知。日志在 `/Users/xiangshu/DailyPaper/scripts/*.log`。

---

## ⚙️ 配置

- Skills 安装位置：`~/.claude/skills/`（_shared / daily-papers / daily-papers-{fetch,review,notes} / daily-papers-weekly / paper-reader / paper-compare / paper-highlights / generate-mocs）
- 主配置：`~/.claude/skills/_shared/user-config.json`
  - 关键词 / 负面词 / 加分词 / arXiv 类目（`cs.SD / eess.AS / cs.CL / cs.MM / cs.HC`）
  - Top-N、git 自动化开关、Obsidian Vault 路径
- 领域术语库：`~/.claude/skills/paper-reader/references/audio-speech-terminology.md`
- 概念分类规则：`~/.claude/skills/paper-reader/references/concept-categories.md`
- 调度脚本：`/Users/xiangshu/DailyPaper/scripts/{daily,weekly}.sh` + `notify.sh` + `archive_old_reports.py`

---

## ⚠️ 已知限制 / 风险

- AI 生成的推荐 / 锐评 / 笔记可能有事实错误或遗漏。**这是工具，不是替代研究判断**
- 自动化用 `claude --dangerously-skip-permissions`，仅在我本机本地运行
- arXiv / HuggingFace 偶尔超时；推送可能在离线时失败（skill 会静默跳过，本地仍有完整 git 历史）
- 国内网络下走 `hf-mirror.com` 镜像
- 领域术语库截至 2026 年初；遇到新概念 paper-reader 会用 WebSearch 补

---

## 📜 License

笔记内容 © 我自己写的部分；论文摘要 / 截图 / 引用归原作者。框架与 skills 部分参考 Apache-2.0 协议的上游项目。
