# 如何重建本草先生 Skill / BUILD

> 本文档说明如何从零开始重建本草先生的 evidence base。如果只是想**使用**本草先生,直接 clone 仓库即可,**不需要重建**。

## 什么时候需要重建

- 想换 LLM 后端(目前用 DeepSeek,可以换 OpenAI / Claude / 本地模型)
- 想加新源(本草图鉴 / 本草书籍 / 思源笔记之外的新资料)
- 想调整蒸馏参数(chunk size、温度、并发)
- 发现 evidence card 有系统性错误,想重新蒸馏

## 重建流程

### 0. 准备

```bash
# 装 Python 3.11+
python --version

# 装 lineage-skill
# 详见 https://github.com/JuneYaooo/lineage-skill
# 装到 ~/.minimax/skills/lineage-skill/
```

### 1. 准备 raw 数据

本草先生要求三个 raw 目录:

```
本草先生/raw/
├── 本草图鉴/            ← 645 个 .md 文件,本草学图录笔记
├── 本草书籍/md/         ← 53 个 .md 文件,古典原典数字化
└── 思源笔记/            ← 个人思源笔记本的 .md 导出
```

(这三个目录是 gitignored,不在 GitHub 上。重建时需要你自己准备或从原出处重新拉取。)

### 2. 打 lineage-skill 补丁

本草先生用的 lineage-skill 有两个补丁,不打会出 6-128 字符的碎块问题或 LLM thinking 卡死。详见 `lineage-skill-patches/README.md`。

```bash
cd ~/.minimax/skills/lineage-skill
# 应用 text_sources.py 和 llm_client.py 的补丁
# 详见 lineage-skill-patches/README.md
```

### 3. 配置 LLM

```bash
# 复制 .env 样例
cp lineage-skill-patches/.env.example ~/.minimax/skills/lineage-skill/.env

# 编辑 .env,填入你的 API key
# LINEAGE_TEXT_API_KEY=sk-...
# LINEAGE_TEXT_BASE_URL=https://api.deepseek.com/v1
# LINEAGE_TEXT_MODEL=deepseek-v4-flash
```

### 4. 跑蒸馏 pipeline

```bash
cd ~/.minimax/skills/lineage-skill
python scripts/run_course_pipeline.py \
  --course-name "本草先生" \
  --raw-dir "C:/Users/hxst01/Downloads/本草先生/本草先生/raw" \
  --output-dir "C:/Users/hxst01/Downloads/本草先生/本草先生/distill-output" \
  --max-chars 80000 \
  --overlap-chars 2000
```

预计耗时:

- Stage 1 (text distillation):2-3 小时(7.4 MB evidence_cards.jsonl,9,406 cards,2 失败,794 成功)
- Stage 2 (course package build):1-2 分钟(40 MB course_package.json)
- Stage 3 (build course skill):1-2 分钟(52 MB references/)

总计约 2.5-3 小时。

### 5. 验证

```bash
# 跑验证
python scripts/run_course_pipeline.py ... --validate

# 看质检报告
cat distill-output/dist/本草先生/validation_report.json
# 应为: {"valid": true, "error_count": 0, "warning_count": 0, ...}
```

### 6. 抽查

蒸馏后,随机抽 5-10 张 evidence card 人工校对:

```bash
# 读一张卡
cat distill-output/dist/本草先生/references/okf/concepts/001-滑石.md
# 验证:
#   - title / description 与原典对得上
#   - resource: lineage://本草先生/concept_xxx
#   - # Citations 段标了出处
```

### 7. 替换 references/

蒸馏完成后,新的 `distill-output/dist/本草先生/references/okf/` 替换本仓库的 `references/okf/` 即可。

```bash
cp -r distill-output/dist/本草先生/references/okf/* \
      本草先生/references/okf/
```

## 蒸馏参数参考

本草先生当前用:

| 参数 | 值 | 备注 |
|------|----|------|
| LLM | `deepseek-v4-flash`(或 `deepseek-chat`) | DeepSeek V3,no thinking |
| max-chars | 80000 | 每 chunk 上限 80K 字符 |
| overlap-chars | 2000 | chunk 之间 2K 字符重叠 |
| temperature | 0.3 | 蒸馏稳定 |
| thinking | disabled | 关 v4-flash 的 thinking |
| concurrency | 1(单线) | 不并行,稳 |

## 故障排查

| 症状 | 原因 | 修复 |
|------|------|------|
| 切出 191,901 个 6-128 字符碎块 | 没打 `text_sources.py` 补丁 | 应用 `chunk_text` 累积段落补丁 |
| 蒸馏卡死、返回 400 thinking 错误 | 没打 `llm_client.py` 补丁 | 应用 thinking 关闭补丁 |
| 2-3 个 chunk 蒸馏失败 | LLM 偶发超时 | 不影响整体,记录在 `text_distillation_quality.json` |
| `references/okf/` 个别文件残缺 | LLM 输出截断 | 跑 `--retry-failed` 重试 |

---

更多 lineage-skill 用法,见 [lineage-skill 官方文档](https://github.com/JuneYaooo/lineage-skill)。
