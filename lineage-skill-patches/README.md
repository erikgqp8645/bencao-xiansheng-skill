# lineage-skill-patches

本草先生蒸馏 lineage-skill 时打的补丁。

## 当前状态(2026-08-20 蒸馏 v6 之后)

**已升级到 lineage-skill 上游版本(GitHub JuneYaooo/lineage-skill,commit `7e2cbc5`)。** 上游已经包含与本目录补丁等效的实现:

| 补丁 | 当前上游对应实现 | 应用后效果 |
|------|----------------|----------|
| `text_sources.py.diff` | `chunk_text` 已有段落缓冲逻辑(见上游 lines 168-198) | 无操作,等效 |
| `llm_client.py.diff` | `call_text_llm` 已是 temperature 0.3 + thinking disabled(见上游 lines 87-89) | 无操作,等效 |

**结论:** 当前 `~/.minimax/skills/lineage-skill/` 已经是正确状态。重新蒸馏时不需要再应用这两个 patch。下面的 diff 文件作为历史记录保留。

## 蒸馏 v1-v5 期间的问题与补丁

本草先生 v1-v5 蒸馏失败,根因有两个:

### 问题 1:chunk_text 把每段切成 6-128 字符的碎块

**症状:** 191,901 个 chunk,平均每 chunk 只有几十个字符。LLM 蒸馏时一个本草条目要跨几十个 chunk,evidence card 之间互相不知道上下文。

**根因:** v0 本地 lineage-skill 的 `chunk_text` 是按段落切分且 `max_chars` 参数形同虚设(没用上)。

**补丁(已包含在上游):** 累积段落直到 `max_chars` 才切。本草先生 v6 用 `max_chars=80000` + `overlap_chars=2000`,产出 **796 chunks**(而非 19 万)。

### 问题 2:deepseek-v4-flash 启用 thinking 卡死

**症状:** LLM 调用超时或返回 400 错误 "thinking not supported"。

**根因:** v0 lineage-skill 默认 temperature=0.8,DeepSeek V4-flash 默认 thinking enabled。

**补丁(已包含在上游):**
- `temperature: 0.3`(更稳的蒸馏)
- `"thinking": {"type": "disabled"}`(关 v4-flash 的 thinking 模式)

本草先生 v6 用 `deepseek-v4-flash` 关 thinking,成功蒸馏 9,406 张 evidence card(2 chunks 失败,794 成功)。

## 当前 lineage-skill 验证

```bash
# 验证 chunk_text 是段落缓冲模式(不是碎块模式)
python -c "
import sys
sys.path.insert(0, r'C:\Users\hxst01\.minimax\skills\lineage-skill\scripts')
from text_sources import chunk_text
text = '\n\n'.join([f'段落{i}' for i in range(1000)])
chunks = chunk_text(text, source_id='x', source_path='x', source_ref='x', max_chars=80000, overlap_chars=2000)
print(f'1000 段 = {len(chunks)} chunk (期望 1)')
print(f'第 0 chunk 长度 = {len(chunks[0][\"text\"])} 字符')
"
# 期望: 1000 段 = 1 chunk,长度 5000 左右

# 验证 llm_client 是 thinking disabled + 温度 0.3
grep -A 3 "temperature" "C:\Users\hxst01\.minimax\skills\lineage-skill\scripts\llm_client.py"
# 期望: "temperature": 0.3, "thinking": {"type": "disabled"}
```

## 重新蒸馏本草先生

如需重建 evidence base(详见 `docs/BUILD.md`):

1. 装 [lineage-skill 上游最新版](https://github.com/JuneYaooo/lineage-skill) 到 `~/.minimax/skills/lineage-skill/`
2. 复制 `.env.example` → `~/.minimax/skills/lineage-skill/.env`,填入 DeepSeek API key
3. 跑 `run_course_pipeline.py`(详见 `docs/BUILD.md`)
4. 不需要再打这两个 patch

## 文件清单

- `text_sources.py.diff` — 历史补丁,等效于上游当前实现
- `llm_client.py.diff` — 历史补丁,等效于上游当前实现
- `.env.example` — DeepSeek API key 配置样例
- `README.md` — 本文件
