# 评测 / Evaluations

本草先生 Skill 的评测方法。

## 当前状态

| 评测维度 | 状态 | 文件 |
|---------|------|------|
| lineage-skill 蒸馏质检 | ✅ 通过 (0 错误, 0 警告) | `validation_report.json` |
| LLM 蒸馏成功率 | ✅ 794/796 chunks (99.75%) | `data/text_distillation/text_distillation_quality.json` |
| evidence card 数量 | 9,406 张 | `references/okf/` |
| 12 OKF 章节覆盖 | ✅ 全覆盖 | `references/okf/index.md` |

## 蒸馏质检 (validation_report.json)

```json
{
  "valid": true,
  "error_count": 0,
  "warning_count": 0,
  "issues": [],
  "required_artifact_count": 6,
  "skill_dir": "."
}
```

## LLM 蒸馏质量 (text_distillation_quality.json)

按源文件统计:每个源本草文献的 chunk 数、成功蒸馏数、失败数、失败原因。

本草先生总计:
- 796 chunks(每 chunk 80K 字符上限)
- 794 成功 → 9,406 evidence cards 平均每 chunk 12 张卡
- 2 失败(LLM 偶发超时/截断,记录在 quality.json)

## 12 OKF 章节覆盖

| 章节 | 文件数 | 主题 |
|------|--------|------|
| concepts | 2,698 | 本草学核心概念(药性/归经/功效) |
| cases | 1,542 | 案例与方剂 |
| methods | 1,352 | 修治 / 炮制 / 配伍 |
| diagnostics | 944 | 诊断与辨证 |
| quotes | 773 | 经典原文引用 |
| boundaries | 1,714 | 禁忌与边界 |
| workflows | 122 | 工作流(诊断→处方) |
| study-paths | 114 | 学习路径 |
| failure-modes | 98 | 失败模式 |
| rubrics | 38 | 评量清单 |
| templates | 11 | 模板 |
| transfer-rules | 6 | 转译规则 |

**合计 9,412 张 evidence card**(实际 9,406 是因为有些章节文件被合并到 index.md)

## 人工抽查建议

随机抽 10 张 evidence card 人工校对:

1. 打开 `references/okf/concepts/<id>-<书名>-<主题>.md`
2. 检查:
   - `title` / `description` 与原典对得上
   - `# Citations` 段标了出处
   - 文字无 LLM 幻觉(描述的是本草学常识,不是凭空捏造)

抽 5 张 cases 卡片:

1. 打开 `references/okf/cases/<id>-<方剂名>.md`
2. 检查方剂组成、剂量、适应症是否符合本草学传统

## 未来评测计划

- [ ] 自动化评测:随机抽 N 张 evidence card,让独立 LLM 评估"是否符合本草学"
- [ ] 蒸馏对比:用不同 LLM(Claude / GPT-4 / DeepSeek V3)蒸馏,对比 evidence card 质量
- [ ] 用户反馈闭环:用户标记"答错"的 card,回流到 review queue

---

评测由本仓库 `data/text_distillation/text_distillation_quality.json` 和 `validation_report.json` 自动产出。
