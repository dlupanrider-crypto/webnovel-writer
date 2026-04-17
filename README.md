# Webnovel Writer

[![License](https://img.shields.io/badge/License-MIT--0-green.svg)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.10+-blue.svg)](https://www.python.org/)
[![OpenClaw](https://img.shields.io/badge/OpenClaw-Skill-orange.svg)](https://openclaw.com)

<a href="https://trendshift.io/repositories/22487" target="_blank"><img src="https://trendshift.io/api/badge/repositories/22487" alt="lingfengQAQ%2Fwebnovel-writer | Trendshift" style="width: 250px; height: 55px;" width="250" height="55"/></a>

## 项目简单介绍

`Webnovel Writer` 是面向 OpenClaw 平台的长篇网文创作 Skill 集合，目标是降低 AI 写作中的"遗忘"和"幻觉"，支持长周期连载创作。包含 8 个 Skills、RAG 检索链路和状态管理。

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

## 快速开始

### 1) 安装依赖

```bash
pip install -r requirements.txt
```

### 2) 初始化小说项目

```bash
/webnovel-init
```

### 3) 配置 RAG 环境（必做）

进入初始化后的书项目根目录，创建 `.env`：

```bash
cp .env.example .env
```

最小配置示例：

```bash
EMBED_BASE_URL=https://api-inference.modelscope.cn/v1
EMBED_MODEL=Qwen/Qwen3-Embedding-8B
EMBED_API_KEY=your_embed_api_key

RERANK_BASE_URL=https://api.jina.ai/v1
RERANK_MODEL=jina-reranker-v3
RERANK_API_KEY=your_rerank_api_key
```

### 4) 开始使用

```bash
/webnovel-plan 1
/webnovel-write 1
/webnovel-review 1-5
```

### 5) 启动可视化面板（可选）

```bash
/webnovel-dashboard
```

## 目录结构

```
webnovel-writer/
├── webnovel-init/          # Skill: 初始化小说项目
├── webnovel-plan/          # Skill: 生成卷级规划
├── webnovel-write/         # Skill: 章节创作（含 Context Agent + Data Agent）
├── webnovel-review/        # Skill: 多维质量审查（含 6 维 Checker）
├── webnovel-query/         # Skill: 查询项目数据
├── webnovel-resume/        # Skill: 中断恢复
├── webnovel-dashboard/     # Skill: 可视化面板
├── webnovel-learn/         # Skill: 提取写作模式
├── scripts/                # Python 脚本（状态管理、RAG、索引、CLI）
├── references/             # 共享参考文件
├── genres/                 # 题材模板
├── templates/              # 输出模板
└── docs/                   # 项目文档
```

## 核心架构

### 防幻觉三定律

| 定律 | 说明 | 执行方式 |
|------|------|---------|
| 大纲即法律 | 遵循大纲，不擅自发挥 | Context Agent 强制加载章节大纲 |
| 设定即物理 | 遵守设定，不自相矛盾 | Consistency Checker 实时校验 |
| 发明需识别 | 新实体必须入库管理 | Data Agent 自动提取并消歧 |

### 写作链路

`/webnovel-write` 流程：Step 1 → 2A(上下文) → 2B(风格转译) → 3(审查) → 4(润色) → 5(数据落盘) → 6(产物输出)
- `--fast`：跳过 2B
- `--minimal`：仅 3 个基础审查

### 六维并行审查

High-point（爽点密度）、Consistency（设定一致性）、Pacing（Strand 比例）、OOC（人设偏离）、Continuity（叙事连贯性）、Reader-pull（钩子强度/追读力）

## 文档

详细文档在 `docs/` 目录：

- 架构与模块：`docs/architecture.md`
- 命令详解：`docs/commands.md`
- RAG 与配置：`docs/rag-and-config.md`
- 题材模板：`docs/genres.md`
- 运维与恢复：`docs/operations.md`
- 文档导航：`docs/README.md`

## 开发

### 运行测试

```bash
pytest webnovel-writer/scripts/data_modules/tests/
```

### 预检

```bash
python -X utf8 "scripts/webnovel.py" --project-root "<PROJECT_ROOT>" preflight
```

## License

MIT-0 (MIT No Attribution). See [LICENSE](LICENSE) for details.
# Webnovel Writer

[![License](https://img.shields.io/badge/License-GPL%20v3-blue.svg)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.10+-blue.svg)](https://www.python.org/)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Compatible-purple.svg)](https://claude.ai/claude-code)

<a href="https://trendshift.io/repositories/22487" target="_blank"><img src="https://trendshift.io/api/badge/repositories/22487" alt="lingfengQAQ%2Fwebnovel-writer | Trendshift" style="width: 250px; height: 55px;" width="250" height="55"/></a>
## 项目简单介绍

`Webnovel Writer` 是基于 Claude Code 的长篇网文创作系统，目标是降低 AI 写作中的“遗忘”和“幻觉”，支持长周期连载创作。

详细文档已拆分到 `docs/`：

- 架构与模块：`docs/architecture.md`
- 命令详解：`docs/commands.md`
- RAG 与配置：`docs/rag-and-config.md`
- 题材模板：`docs/genres.md`
- 运维与恢复：`docs/operations.md`
- 文档导航：`docs/README.md`

## 快速开始

### 1) 安装插件（官方 Marketplace）

```bash
claude plugin marketplace add lingfengQAQ/webnovel-writer --scope user
claude plugin install webnovel-writer@webnovel-writer-marketplace --scope user
```

> 仅当前项目生效时，将 `--scope user` 改为 `--scope project`。

### 2) 安装 Python 依赖

```bash
python -m pip install -r https://raw.githubusercontent.com/lingfengQAQ/webnovel-writer/HEAD/requirements.txt
```

说明：该入口会同时安装核心写作链路与 Dashboard 依赖。

### 3) 初始化小说项目

在 Claude Code 中执行：

```bash
/webnovel-init
```

说明：`/webnovel-init` 会在当前 Workspace 下按书名创建 `PROJECT_ROOT`（子目录），并在 `workspace/.claude/.webnovel-current-project` 写入当前项目指针。

### 4) 配置 RAG 环境（必做）

进入初始化后的书项目根目录，创建 `.env`：

```bash
cp .env.example .env
```

最小配置示例：

```bash
EMBED_BASE_URL=https://api-inference.modelscope.cn/v1
EMBED_MODEL=Qwen/Qwen3-Embedding-8B
EMBED_API_KEY=your_embed_api_key

RERANK_BASE_URL=https://api.jina.ai/v1
RERANK_MODEL=jina-reranker-v3
RERANK_API_KEY=your_rerank_api_key
```

### 5) 开始使用

```bash
/webnovel-plan 1
/webnovel-write 1
/webnovel-review 1-5
```

如需排查本地 CLI / 插件目录 / 项目根解析问题，可直接运行统一预检：

```bash
python -X utf8 "<CLAUDE_PLUGIN_ROOT>/scripts/webnovel.py" --project-root "<WORKSPACE_ROOT>" preflight
```

### 6) 启动可视化面板（可选）

```bash
/webnovel-dashboard
```

说明：
- Dashboard 为只读面板（项目状态、实体图谱、章节/大纲浏览、追读力查看）。
- 前端构建产物已随插件发布，使用者无需本地 `npm build`。

### 7) Agent 模型设置（可选）

本项目所有内置 Agent 默认配置为：

```yaml
model: inherit
```

表示子 Agent 继承当前 Claude 会话所用模型。

如果要单独给某个 Agent 指定模型，编辑对应文件（`webnovel-writer/agents/*.md`）的 frontmatter，例如：

```yaml
---
name: context-agent
description: ...
tools: Read, Grep, Bash
model: sonnet
---
```

常见可选值：`inherit` / `sonnet` / `opus` / `haiku`（以 Claude Code 当前支持为准）。

## 更新简介

| 版本 | 说明 |
|------|------|
| **v5.5.4 (当前)** | 补齐写作链提示词强约束（流程硬约束、中文思维写作约束、Step 职责边界）；统一中文化审查/润色/Agent 报告文案；清理文档内部版本号与版本历史，降低与插件发版版本混淆。 |
| **v5.5.3** | 新增统一 `preflight` 预检命令；写作链 CLI 示例统一为 UTF-8 运行方式，收口文档中的长 shell 预检片段并降低 Windows 终端乱码风险。 |
| **v5.5.2** | 支持将详细大纲中的章节名同步到正文文件名；修复 workflow_manager 在无参 find_project_root monkeypatch 下的兼容性问题。 |
| **v5.5.1** | 修复卷级单文件大纲在上下文快照中的章节提取问题；补齐命令文档中遗漏的 `/webnovel-dashboard` 与 `/webnovel-learn`。 |
| **v5.5.0** | 新增只读可视化 Dashboard Skill（`/webnovel-dashboard`）与实时刷新能力；支持插件目录启动与预构建前端分发 |
| **v5.4.4** | 引入官方 Plugin Marketplace 安装机制；统一修复 Skills/Agents/References 的 CLI 调用（`CLAUDE_PLUGIN_ROOT` 单路径，透传命令统一 `--`） |
| **v5.4.3** | 增强智能 RAG 上下文辅助（`auto/graph_hybrid` 回退 BM25） |
| **v5.3** | 引入追读力系统（Hook / Cool-point / 微兑现 / 债务追踪） |

## 插件发版

推荐使用 GitHub Actions 的 `Plugin Release` 工作流统一发版：

1. 先在本地同步版本信息：
   ```bash
   python -X utf8 webnovel-writer/scripts/sync_plugin_version.py --version 5.5.4 --release-notes "本次版本说明"
   ```
2. 提交并推送版本变更（`README.md`、`plugin.json`、`marketplace.json`）。
3. 打开仓库的 Actions 页面，选择 `Plugin Release`。
4. 输入与当前仓库元数据一致的 `version`（例如 `5.5.4`）和用于 GitHub Release 的 `release_notes`。
5. 工作流会执行以下动作：
   - 校验 `plugin.json`、`marketplace.json` 与 README 当前版本已经一致
   - 校验当前版本与输入的 `version` 一致
   - 创建并推送 `vX.Y.Z` Tag
   - 创建同名 GitHub Release

日常开发中，`Plugin Version Check` 会在 Push / PR 时自动校验版本信息是否一致。

## 开源协议
本项目使用 `GPL v3` 协议，详见 `LICENSE`。

## Star 历史

[![Star History Chart](https://api.star-history.com/svg?repos=lingfengQAQ/webnovel-writer&type=Date)](https://star-history.com/#lingfengQAQ/webnovel-writer&Date)

## 致谢

本项目使用 **Claude Code + Gemini CLI + Codex** 配合 Vibe Coding 方式开发。  
灵感来源：[Linux.do 帖子](https://linux.do/t/topic/1397944/49)

## 贡献

欢迎提交 Issue 和 PR：

```bash
git checkout -b feature/your-feature
git commit -m "feat: add your feature"
git push origin feature/your-feature
```
