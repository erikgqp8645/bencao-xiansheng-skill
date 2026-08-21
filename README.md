# 本草先生 (Mr. Ben Cao)

> 本草学文献整理 Skill — source-grounded references to classical Chinese materia medica.

[![GitHub](https://img.shields.io/badge/GitHub-bencao--xiansheng--skill-blue)](https://github.com/erikgqp8645/bencao-xiansheng-skill)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Evidence cards](https://img.shields.io/badge/evidence-9%2C406-green)](references/okf/)

本草先生是一个**本草学文献整理 Skill**。它把本草图鉴(645 文件)、本草书籍 md(53 文件)、思源笔记共约 700 个 .md 源文件,蒸馏成 **9,406 张结构化 evidence card**,按 12 个 OKF 章节组织(本草概念 / 经典原文 / 案例方剂 / 修治方法 / 诊断辨证 / 禁忌边界 / 评量 / 模板 / 工作流 / 失败模式 / 转译规则 / 学习路径)。

本草先生不是医生、不出具医疗建议,只做本草文献的整理与查证。

## 快速开始

### 适用平台

- [MiniMax Code / Mavis](https://github.com/MiniMax-ai/minimax-code) — `~/.minimax/skills/本草先生/`
- Claude Code / Cursor / OpenClaw — 通过 `agents/openai.yaml` 或 `agents/openclaw.yaml` 加载
- 任何支持 Anthropic / OpenAI tool_use 协议的 Agent — 复制 `SKILL.md` + `references/` + `scripts/`

### 加载 Skill

把本仓库 clone 到你的 skills 目录,然后告诉 agent "按本草先生 skill 回答":

```bash
# MiniMax Code / Mavis
git clone https://github.com/erikgqp8645/bencao-xiansheng-skill.git \
  ~/.minimax/skills/本草先生
```

或者直接告诉 agent:

> "请用本草先生 skill 帮我查一味药:茯苓的性味、归经、功效、修治。"

### 4 角色

| 角色 | 英文 | 适用场景 |
|------|------|---------|
| mentor (默认) | 导师 | 系统学习本草学,从《神农本草经》上品开始逐章带读 |
| expert | 专家 | 学术查证,考据异同,对比诸家说法 |
| consultant | 顾问 | 文献整理,主题综述,对照表 |
| practitioner | 临床师 | 从临床应用反查本草学传统记载(不替代医师诊断) |

切换方式:在提问时指明角色,如"用 expert 模式查《本草纲目》滑石条目各家说法"。

## 仓库结构

```
本草先生/                                    (本仓库根)
├── README.md                                ← 本文件
├── SKILL.md                                 ← Skill 主入口(给 LLM 看的 prompt + 行为)
├── lineage_manifest.json                    ← lineage-skill 生成元数据
├── 本草先生-需求文档.md                      ← v0.2 项目需求规格
│
├── docs/                                    ← 文档
│   ├── USE_AND_RISK_NOTICE.md               ← 使用与风险声明
│   ├── BUILD.md                             ← 如何重新蒸馏
│   └── INSTALL.md                           ← 详细安装指南
│
├── evals/                                   ← 评测
│   ├── README.md                            ← 评测方法
│   └── validation_report.json               ← 蒸馏质检报告(0 错误)
│
├── references/                              ← 9,406 张 evidence card + 索引
│   ├── okf/                                 ← 12 章节 OKF 知识库
│   │   ├── index.md                         ←   12 章节索引
│   │   ├── concepts/ (2,698)                ←   本草学概念卡
│   │   ├── cases/ (1,542)                   ←   案例与方剂卡
│   │   ├── methods/ (1,352)                 ←   修治 / 炮制 / 配伍卡
│   │   ├── diagnostics/ (944)               ←   诊断与辨证卡
│   │   ├── boundaries/ (1,714)              ←   禁忌与边界卡
│   │   ├── quotes/ (773)                    ←   经典原文引用
│   │   ├── workflows/ (122)                 ←   工作流卡
│   │   ├── study-paths/ (114)               ←   学习路径卡
│   │   ├── failure-modes/ (98)              ←   失败模式
│   │   ├── rubrics/ (38)                    ←   评量清单
│   │   ├── templates/ (11)                  ←   模板
│   │   └── transfer-rules/ (6)              ←   转译规则
│   └── evidence_map.json                    ← 证据 ↔ 概念链接 (6 MB)
│
├── scripts/                                 ← Skill 配套工具
│   ├── search_course_notes.py               ←   关键词搜索本草学内容
│   └── fetch_course_evidence.py             ←   按 ID 取出完整证据链
│
├── agents/                                  ← Agent 平台适配
│   ├── openai.yaml                          ←   OpenAI / OpenClaw 平台 manifest
│   └── openclaw.yaml                        ←   OpenClaw 平台 manifest(带 trust surface)
│
├── data/                                    ← 蒸馏原始产物(.gitignore 大文件)
│   ├── text_sources/
│   │   ├── chunks.jsonl (47 MB)             ←   蒸馏输入 chunks
│   │   └── source_manifest.json             ←   chunk 元数据
│   ├── text_distillation/
│   │   ├── evidence_cards.jsonl (7.6 MB)    ←   LLM 蒸馏的 evidence card
│   │   ├── source_summaries.json            ←   源文档 summary
│   │   ├── text_course_synthesis.md         ←   整书 synthesis
│   │   └── text_distillation_quality.json   ←   蒸馏质量报告
│   ├── course_package.json (40 MB)          ← 整合后的最终包
│   └── course_package_build_report.json     ← 整合 build 报告
│
├── lineage-skill-patches/                   ← 蒸馏 lineage-skill 时打的补丁
│   ├── README.md                            ←   怎么打补丁
│   ├── text_sources.py.diff                 ←   chunk_text 累积段落切分
│   ├── llm_client.py.diff                   ←   关闭 thinking + 温度 0.3
│   └── .env.example                         ←   DeepSeek API key 配置样例
│
└── assets/                                  ← 图标、占位资源
    └── README.md                            ← 暂空
```

## 数据来源

本草先生的 evidence base 来自三个一手源:

1. **本草图鉴**(`本草图鉴/`)—— 645 个 .md 文件,含神农本草经、各家本草图录
2. **本草书籍 md**(`本草书籍/md/`)—— 53 个 .md 文件,古典原典数字化
3. **思源笔记**(`思源笔记/`)—— 用户个人本草学笔记,作为补充语料

源数据约 46 MB,共 701 个 .md 文件,经去图引用 + 切块(796 chunks) + LLM 蒸馏(9,406 cards)而成。

## 重建 evidence base

如果需要从零重建(例如想换 LLM、加新源、调参数):

1. 装 [lineage-skill](https://github.com/JuneYaooo/lineage-skill) 到 `~/.minimax/skills/lineage-skill/`
2. 应用本仓库 `lineage-skill-patches/` 下的补丁(详见 `lineage-skill-patches/README.md`)
3. 准备 `raw/`(本草图鉴 + 本草书籍 + 思源笔记)
4. 跑 `run_course_pipeline.py` — 详见 `docs/BUILD.md`

不重建也没关系:`references/okf/` 已经包含 9,406 张卡,直接用。

## 使用与风险

本草先生是**本草文献整理者**,不是医生,不出具医疗建议。所有本草学讨论与临床实践分开。

使用前请阅读 **[docs/USE_AND_RISK_NOTICE.md](docs/USE_AND_RISK_NOTICE.md)**。

## 贡献

本 Skill 为个人学习用途整理,语料来自公开本草学经典 + 个人笔记。

如果你想引用本草先生的内容,请标注:"本草先生 · 本草文献整理"。

## 致谢

- 蒸馏框架: [JuneYaooo/lineage-skill](https://github.com/JuneYaooo/lineage-skill)
- LLM 后端: DeepSeek API (deepseek-chat / deepseek-v4-flash)
- 数据来源:本草图鉴 + 本草书籍 + 思源笔记
