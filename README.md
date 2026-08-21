# 本草先生

> Source-grounded AI Skill for Traditional Chinese Medicine herbalism (本草学), built from 51 ancient materia medica texts via the lineage-skill distillation pipeline.

## 这是什么

本草先生是一个 **Hermes Agent Skill**,用 lineage-skill 蒸馏管线从 51 本本草古籍 + 思源笔记的中草药图鉴(701 个文档,46 MB)中蒸馏出来的本草学知识包。

- **规模**: 9,406 张 evidence cards(LLM 蒸馏)
- **结构**: SKILL.md 入口 + references/okf/ 按 concepts/methods/diagnostics/cases/quotes/rubrics/failure-modes 等分类
- **覆盖**: 历代本草原文引用 772 条,概念 2,697,方法 1,351,案例 943,边界(使用禁忌)1,713,流程(方剂)121,评估 37
- **典型用法**:
  - 问"人参有什么功效"→ 返回本草纲目/本草述校注原文+出处
  - 问"如何炮制半夏"→ 返回操作清单+风险点
  - 问"我气虚想吃点什么药"→ 返回多家立场对比+临床建议

## 蒸馏结果

完整 Skill 在 `distill-output-v6/dist/本草先生/`,52 MB。

- `SKILL.md` — 技能入口,触发词 + 角色路由
- `references/okf/` — 知识库(分 12 个子目录)
- `scripts/` — 辅助脚本(fetch/search)
- `lineage_manifest.json` — 元数据
- `validation_report.json` — 验证报告

中间产物在 `distill-output-v6/本草先生/`(text_distillation/、text_sources/、lineage_progress.json、course_package.json),用于复现和审计。

## 蒸馏管线(lineage-skill)需要打的两个补丁

`lineage-skill` 默认的 chunker 做的是**段落级切分**(本草每段就是一句话,导致 191,901 个 LLM call,要跑几天)。需要改:

### 1. `text_sources.py` — `chunk_text` 函数

**症状**: 191,901 个 6-128 字符的"chunk",`max_chars` 被无视。
**修复**: 累积段落直到达到 `max_chars` 才切,而不是每段一个 chunk。

```diff
-    for start, end, value in paragraph_spans(text):
-        for piece_start, piece_end, piece in split_long_span(start, value, max_chars, overlap_chars):
-            ...
+    paragraphs = paragraph_spans(text)
+    buf_text = ""
+    buf_start = None
+    buf_end = 0
+    for p_start, p_end, p_value in paragraphs:
+        if len(p_value) > max_chars:
+            # flush buffer, split long paragraph
+            ...
+        elif buf_text and len(buf_text) + 2 + len(p_value) > max_chars:
+            # flush current buffer
+            ...
+        else:
+            # accumulate
+            ...
```

**效果**: 同样数据从 191,901 chunks 降到 ~796 chunks(46 MB / 80K chars)。

### 2. `llm_client.py` — `call_text_llm` 函数

**症状 1**: 默认调用 v3 模型,context 64K,治标不治本。
**修复**: 用 `deepseek-v4-flash`(1M context),关闭 thinking 模式(默认开,会拖慢 5-10x):

```diff
                 json={
                     "model": model,
                     "messages": messages,
-                    "temperature": 0.8,
+                    "temperature": 0.3,
                     "max_tokens": max_tokens,
+                    "thinking": {"type": "disabled"},
                 },
```

`.env` 配置(放在 `lineage-skill/` 下):
```env
LINEAGE_TEXT_API_KEY=sk-...
LINEAGE_TEXT_BASE_URL=https://api.deepseek.com/v1
LINEAGE_TEXT_MODEL=deepseek-v4-flash
DISTILL_USE_LLM=1
```

## 复现蒸馏

```bash
# 1. 准备 raw/ 数据(从思源笔记 + GitHub 公开仓库)
#    - 中草药图鉴 思源笔记本 20240205144800-g67i349
#    - 51 本 GitHub 本草古籍 xiaopangxia/TCM-Ancient-Books

# 2. 应用两个补丁到 lineage-skill

# 3. 启动蒸馏(用 v6 的参数)
python lineage-skill/scripts/distill_text_course.py \
  --course-name 本草先生 \
  --base-dir ./distill-output-v6 \
  --max-chars 80000 \
  --overlap-chars 2000 \
  --input ./raw/本草书籍/md \
  --input ./raw/中草药图鉴

# 4. 出 skill
python lineage-skill/scripts/build_course_skill.py \
  --course-name 本草先生 --skill-name 本草先生 \
  --mode custom --apprenticeship full --learner-state external \
  --source-dir ./distill-output-v6/本草先生 \
  --output-dir ./distill-output-v6/dist
```

## 项目结构

```
本草先生/
├── README.md                          ← 本文件
├── 本草先生-需求文档.md                ← 项目规格说明
├── .gitignore
├── raw/                                ← [gitignored] 源数据(思源笔记 + GitHub)
├── distill-output/                    ← [gitignored] 试错过程
├── distill-output-v1/ ~ v5/            ← [gitignored] 5 次失败重试
└── distill-output-v6/                  ← 最终蒸馏(本次提交)
    ├── text_sources/                   ← 切块源数据(796 chunks)
    ├── text_distillation/              ← 9,406 evidence cards + 质量报告
    ├── lineage_progress.json
    ├── course_package.json             ← 40 MB 聚合包
    └── dist/本草先生/                   ← **THE SKILL** (52 MB)
```

## 关键文档

- `本草先生-需求文档.md` — v0.2 项目规格,包括 51 本古籍清单、教师模型设计、流水线映射、验收标准
- `distill-output-v6/本草先生/text_distillation/text_distillation_quality.json` — 蒸馏质量报告
- `distill-output-v6/本草先生/dist/本草先生/SKILL.md` — 最终 skill 入口

## 致谢

- 数据源:思源笔记「中草药图鉴」笔记本 + 51 本本草古籍(GitHub 公开仓库 `xiaopangxia/TCM-Ancient-Books`)
- 蒸馏框架:lineage-skill(`erikgqp8645/lineage-skill`)
- LLM:DeepSeek v4-flash(1M 上下文,关闭 thinking)
