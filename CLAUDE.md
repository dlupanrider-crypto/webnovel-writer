# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with this repository.

## 项目简介

`Webnovel Writer` 是基于 Claude Code 的长篇网文创作系统，目标是降低 AI 写作中的"遗忘"和"幻觉"，支持长周期连载创作。核心是一个 Claude Code 插件，包含 7 个 Skills、8 个 Agents、RAG 检索链路和状态管理。

## 目录结构

```
webnovel-writer/              # 插件根目录（CLAUDE_PLUGIN_ROOT）
├── skills/                   # 7 个 Skill：init/plan/write/review/query/resume/learn/dashboard
├── agents/                   # 8 个 Agent：context-agent/data-agent + 6 维 Checker
├── genres/                   # 题材模板（狗血言情/现实向/玄幻/悬疑推理/知乎短/宫斗）
├── references/               # 共享参考文件（体裁 profile、阅读力分类学等）
├── scripts/                  # Python 脚本（状态管理、RAG、索引、CLI）
│   └── data_modules/         # 数据层模块和测试
├── dashboard/                # 可视化面板（Flask + 前端构建产物）
└── .claude-plugin/           # 插件元数据
docs/                         # 项目文档（架构、命令、RAG、运维）
```

## 开发命令

### 安装依赖

```bash
pip install -r requirements.txt
```

### 运行测试

```bash
# 运行全部测试
pytest webnovel-writer/scripts/data_modules/tests/

# 运行单个测试文件
pytest webnovel-writer/scripts/data_modules/tests/test_context_manager.py

# 带覆盖率
pytest --cov=webnovel-writer/scripts/data_modules webnovel-writer/scripts/data_modules/tests/
```

### 插件发版

```bash
# 1. 同步版本信息
python -X utf8 webnovel-writer/scripts/sync_plugin_version.py --version X.Y.Z --release-notes "说明"

# 2. 提交并推送，然后通过 GitHub Actions 触发 Plugin Release
```

### 预检命令

```bash
python -X utf8 "<CLAUDE_PLUGIN_ROOT>/scripts/webnovel.py" --project-root "<WORKSPACE_ROOT>" preflight
```

## 核心架构

### 防幻觉三定律

| 定律 | 说明 | 执行方式 |
|------|------|---------|
| 大纲即法律 | 遵循大纲，不擅自发挥 | Context Agent 强制加载章节大纲 |
| 设定即物理 | 遵守设定，不自相矛盾 | Consistency Checker 实时校验 |
| 发明需识别 | 新实体必须入库管理 | Data Agent 自动提取并消歧 |

### 双 Agent 架构

- **Context Agent（读）**：写作前构建"创作任务书"，提供上下文、约束和追读力策略
- **Data Agent（写）**：从正文提取实体与状态变化，更新 `state.json`、`index.db`、`vectors.db`

### 六维并行审查

High-point（爽点密度）、Consistency（设定一致性）、Pacing（Strand 比例）、OOC（人设偏离）、Continuity（叙事连贯性）、Reader-pull（钩子强度/追读力）

### 写作链路

`/webnovel-write` 流程：Step 1 → 2A(上下文) → 2B(风格转译) → 3(审查) → 4(润色) → 5(数据落盘) → 6(产物输出)
- `--fast`：跳过 2B
- `--minimal`：仅 3 个基础审查

### RAG 检索

```
查询 → QueryRouter(auto) → vector / bm25 / hybrid / graph_hybrid
                     └→ RRF 融合 + Rerank → Top-K
```

环境变量加载顺序：进程环境变量 > 书项目 `.env` > 用户级 `~/.claude/webnovel-writer/.env`

### 状态存储

- `.webnovel/state.json`：项目状态（进度、章节元数据、实体关系）
- `index.db`：SQLite 索引（审查指标、实体、阅读力追踪）
- `vectors.db`：向量存储

## Skill 列表

| Skill | 用途 |
|-------|------|
| `/webnovel-init` | 初始化小说项目（深度模式，分阶段收集创作信息） |
| `/webnovel-plan [卷号]` | 生成卷级规划与章节大纲 |
| `/webnovel-write [章号]` | 执行完整章节创作流程 |
| `/webnovel-review [范围]` | 多维质量审查 |
| `/webnovel-query [关键词]` | 查询角色/伏笔/节奏/状态等 |
| `/webnovel-resume` | 中断后自动恢复 |
| `/webnovel-dashboard` | 只读可视化面板 |
| `/webnovel-learn [内容]` | 提取可复用写作模式到项目记忆 |

## 重要约定

- 所有 Skill 文档在对应 `skills/*/SKILL.md` 中，引用加载采用分级策略（L0/L1/L2/L3）
- `references/shared/` 是单一事实源，遇到 `<!-- DEPRECATED:` 标记的文件一律跳过
- 子 Agent 默认配置 `model: inherit`（继承主会话模型），可在 `agents/*.md` frontmatter 中覆盖
- Python 脚本统一使用 `python -X utf8` 运行，避免 Windows 终端编码问题
- 插件版本信息需保持三处一致：`plugin.json`、`marketplace.json`、`README.md`
