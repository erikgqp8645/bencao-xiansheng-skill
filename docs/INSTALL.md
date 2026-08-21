# 安装指南 / INSTALL

本草先生 Skill 兼容多种 Agent 平台。

## 平台 1: MiniMax Code / Mavis(推荐)

```bash
# 1. Clone 仓库
git clone https://github.com/erikgqp8645/bencao-xiansheng-skill.git \
  ~/.minimax/skills/本草先生

# 2. 验证
ls ~/.minimax/skills/本草先生/SKILL.md
ls ~/.minimax/skills/本草先生/references/okf/index.md
```

启动 MiniMax Code,本草先生 skill 会自动出现在可用 skills 列表里。

## 平台 2: Claude Code / Cursor / OpenClaw

这些平台用 `agents/openai.yaml` 或 `agents/openclaw.yaml` 加载。

### Claude Code(Anthropic 协议)

```bash
# 把 SKILL.md + references/ + scripts/ 复制到 Claude 的 skills 目录
# (具体路径参考 Claude Code 文档)
cp SKILL.md /path/to/claude/skills/本草先生.md
cp -r references/ /path/to/claude/skills/本草先生/
cp -r scripts/ /path/to/claude/skills/本草先生/
```

### OpenClaw / OpenAI 协议

```bash
# 用 openclaw.yaml 作为 manifest
cp agents/openclaw.yaml /path/to/openclaw/agents/本草先生.yaml
cp -r references/ /path/to/openclaw/agents/本草先生/
cp -r scripts/ /path/to/openclaw/agents/本草先生/
```

启动 OpenClaw agent,本草先生会自动注册。

## 平台 3: 通用 Anthropic / OpenAI 协议

如果你的 Agent 支持自定义 skill / tool:

1. 把 `SKILL.md` 作为 system prompt
2. 把 `references/okf/index.md` 作为入口导航
3. 把 `scripts/search_course_notes.py` 作为可用 tool
4. 把 `scripts/fetch_course_evidence.py` 作为可用 tool

## 验证安装

### 用 search 脚本

```bash
cd ~/.minimax/skills/本草先生
python scripts/search_course_notes.py "茯苓"
# 应输出:多条 evidence card 命中,含 card_id / title / source_ref
```

### 用 fetch 脚本

```bash
cd ~/.minimax/skills/本草先生/scripts
python fetch_course_evidence.py --card-id concept_xxxx --references-dir ../references
# 应输出:完整 evidence chain(JSON)
```

### 让 Agent 用本草先生 skill 回答

告诉你的 agent:

> "请用本草先生 skill 帮我查一味药:滑石的性味归经和各家说法对比。"

如果 skill 加载成功,Agent 会从 `references/okf/concepts/` 找到滑石相关卡片,引用《神农本草经》《本草衍义》等原典,按 expert 模式给出对比。

## 升级

```bash
cd ~/.minimax/skills/本草先生
git pull origin master
```

## 卸载

```bash
rm -rf ~/.minimax/skills/本草先生
```

---

更多问题看 [USE_AND_RISK_NOTICE.md](USE_AND_RISK_NOTICE.md) 和 [BUILD.md](BUILD.md)。
