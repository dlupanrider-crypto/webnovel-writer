---
name: webnovel-write
description: Use when writing web novel chapters. Executes the complete writing workflow including context gathering, drafting, multi-dimensional review, polishing, and data extraction.
version: 5.5.4
metadata:
  openclaw:
    requires:
      env:
        - OPENAI_API_KEY
      bins:
        - python3
    emoji: "✍️"
    homepage: https://github.com/lingfengQAQ/webnovel-writer
---

# Chapter Writing (Structured Workflow)

## 目标

- 以稳定流程产出可发布章节：优先使用 `正文/第{NNNN}章-{title_safe}.md`，无标题时回退 `正文/第{NNNN}章.md`。
- 默认章节字数目标：2000-2500（用户或大纲明确覆盖时从其约定）。
- 保证审查、润色、数据回写完整闭环，避免“写完即丢上下文”。
- 输出直接可被后续章节消费的结构化数据：`review_metrics`、`summaries`、`chapter_meta`。

## 执行原则

1. 先校验输入完整性，再进入写作流程；缺关键输入时立即阻断。
2. 审查与数据回写是硬步骤，`--fast`/`--minimal` 只允许降级可选环节。
3. 参考资料严格按步骤按需加载，不一次性灌入全部文档。
4. Step 2B 与 Step 4 职责分离：2B 只做风格转译，4 只做问题修复与质控。
5. 任一步失败优先做最小回滚，不重跑全流程。

## 模式定义

- `/webnovel-write`：Step 1 → 2A → 2B → 3 → 4 → 5 → 6
- `/webnovel-write --fast`：Step 1 → 2A → 3 → 4 → 5 → 6（跳过 2B）
- `/webnovel-write --minimal`：Step 1 → 2A → 3（仅3个基础审查）→ 4 → 5 → 6

最小产物（所有模式）：
- `正文/第{NNNN}章-{title_safe}.md` 或 `正文/第{NNNN}章.md`
- `index.db.review_metrics` 新纪录（含 `overall_score`）
- `.webnovel/summaries/ch{NNNN}.md`
- `.webnovel/state.json` 的进度与 `chapter_meta` 更新

### 流程硬约束（禁止事项）

- **禁止并步**：不得将两个 Step 合并为一个动作执行（如同时做 2A 和 3）。
- **禁止跳步**：不得跳过未被模式定义标记为可跳过的 Step。
- **禁止临时改名**：不得将 Step 的输出产物改写为非标准文件名或格式。
- **禁止自创模式**：`--fast` / `--minimal` 只允许按上方定义裁剪步骤，不允许自创混合模式、"半步"或"简化版"。
- **禁止自审替代**：Step 3 审查必须由 Task 子代理执行，主流程不得内联伪造审查结论。
- **禁止源码探测**：脚本调用方式以本文档与 data-agent 文档中的命令示例为准，命令失败时查日志定位问题，不去翻源码学习调用方式。

## 引用加载等级（strict, lazy）

- L0：未进入对应步骤前，不加载任何参考文件。
- L1：每步仅加载该步“必读”文件。
- L2：仅在触发条件满足时加载“条件必读/可选”文件。

路径约定：
- `references/...` 相对当前 skill 目录。
- `../../references/...` 指向全局共享参考。

## References（逐文件引用清单）

### 根目录

- `references/step-3-review-gate.md`
  - 用途：Step 3 审查调用模板、汇总格式、落库 JSON 规范。
  - 触发：Step 3 必读。
- `references/step-5-debt-switch.md`
  - 用途：Step 5 债务利息开关规则（默认关闭）。
  - 触发：Step 5 必读。
- `../../references/shared/core-constraints.md`
  - 用途：Step 2A 写作硬约束（大纲即法律 / 设定即物理 / 发明需识别）。
  - 触发：Step 2A 必读。
- `references/polish-guide.md`
  - 用途：Step 4 问题修复、Anti-AI 与 No-Poison 规则。
  - 触发：Step 4 必读。
- `references/writing/typesetting.md`
  - 用途：Step 4 移动端阅读排版与发布前速查。
  - 触发：Step 4 必读。
- `references/style-adapter.md`
  - 用途：Step 2B 风格转译规则，不改剧情事实。
  - 触发：Step 2B 执行时必读（`--fast`/`--minimal` 跳过）。
- `references/style-variants.md`
  - 用途：Step 1（内置 Contract）开头/钩子/节奏变体与重复风险控制。
  - 触发：Step 1 当需要做差异化设计时加载。
- `../../references/reading-power-taxonomy.md`
  - 用途：Step 1（内置 Contract）钩子、爽点、微兑现 taxonomy。
  - 触发：Step 1 当需要追读力设计时加载。
- `../../references/genre-profiles.md`
  - 用途：Step 1（内置 Contract）按题材配置节奏阈值与钩子偏好。
  - 触发：Step 1 当 `state.project.genre` 已知时加载。
- `references/writing/genre-hook-payoff-library.md`
  - 用途：电竞/直播文/克苏鲁的钩子与微兑现快速库。
  - 触发：Step 1 题材命中 `esports/livestream/cosmic-horror` 时必读。

### writing（问题定向加读）

- `references/writing/combat-scenes.md`
  - 触发：战斗章或审查命中“战斗可读性/镜头混乱”。
- `references/writing/dialogue-writing.md`
  - 触发：审查命中 OOC、对话说明书化、对白辨识差。
- `references/writing/emotion-psychology.md`
  - 触发：情绪转折生硬、动机断层、共情弱。
- `references/writing/scene-description.md`
  - 触发：场景空泛、空间方位不清、切场突兀。
- `references/writing/desire-description.md`
  - 触发：主角目标弱、欲望驱动力不足。

## 工具策略（按需）

- `Read/Grep`：读取 `state.json`、大纲、章节正文与参考文件。
- `Bash`：运行 `extract_chapter_context.py`、`index_manager`、`workflow_manager`。
- `Task`：调用 `context-agent`、审查 subagent、`data-agent` 并行执行。

## 交互流程

### Step 0：预检与上下文最小加载

必须做：
- 解析真实书项目根（book project_root）：必须包含 `.webnovel/state.json`。
- 校验核心输入：`大纲/总纲.md`、`${SCRIPTS_DIR}/extract_chapter_context.py` 存在。
  - 规范化变量：
    - `WORKSPACE_ROOT`：工作区根目录（可能是书项目的父目录）
    - `PROJECT_ROOT`：真实书项目根目录（必须包含 `.webnovel/state.json`）
    - `SKILL_ROOT`：skill 所在目录（当前 skill 根目录）
    - `SCRIPTS_DIR`：脚本目录（`${SKILL_ROOT}/../scripts`）
    - `chapter_num`：当前章号（整数）
    - `chapter_padded`：四位章号（如 `0007`）

环境设置（bash 命令执行前）：
```bash
export WORKSPACE_ROOT="${PROJECT_DIR:-$PWD}"
export SKILL_ROOT="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
export SCRIPTS_DIR="${SKILL_ROOT}/../scripts"

python -X utf8 "${SCRIPTS_DIR}/webnovel.py" --project-root "${WORKSPACE_ROOT}" preflight
export PROJECT_ROOT="$(python -X utf8 "${SCRIPTS_DIR}/webnovel.py" --project-root "${WORKSPACE_ROOT}" where)"
```

**硬门槛**：`preflight` 必须成功。它统一校验 `SKILL_ROOT` 派生出的 `SCRIPTS_DIR`、`webnovel.py`、`extract_chapter_context.py` 和解析出的 `PROJECT_ROOT`。任一失败都立即阻断。

输出：
- “已就绪输入”与“缺失输入”清单；缺失则阻断并提示先补齐。

### Step 0.5：工作流断点记录（best-effort，不阻断）

```bash
python -X utf8 "${SCRIPTS_DIR}/webnovel.py" --project-root "${PROJECT_ROOT}" workflow start-task --command webnovel-write --chapter {chapter_num} || true
python -X utf8 "${SCRIPTS_DIR}/webnovel.py" --project-root "${PROJECT_ROOT}" workflow start-step --step-id "Step 1" --step-name "Context Agent" || true
python -X utf8 "${SCRIPTS_DIR}/webnovel.py" --project-root "${PROJECT_ROOT}" workflow complete-step --step-id "Step 1" --artifacts '{"ok":true}' || true
python -X utf8 "${SCRIPTS_DIR}/webnovel.py" --project-root "${PROJECT_ROOT}" workflow complete-task --artifacts '{"ok":true}' || true
```

要求：
- `--step-id` 仅允许：`Step 1` / `Step 2A` / `Step 2B` / `Step 3` / `Step 4` / `Step 5` / `Step 6`。
- 任何记录失败只记警告，不阻断写作。
- 每个 Step 执行结束后，同样需要 `complete-step`（失败不阻断）。

### Step 1：Context Agent（内置 Context Contract，生成直写执行包）

> **Role**: 创作执行包生成器。目标是"能直接开写"，不堆信息。
> **Philosophy**: 按需召回 + 推断补全，确保接住上章、场景清晰、留出钩子。

**输入参数**：
```json
{
  "chapter": {chapter_num},
  "project_root": "${PROJECT_ROOT}",
  "storage_path": ".webnovel/",
  "state_file": ".webnovel/state.json"
}
```

**核心参考**：
- Taxonomy: `references/reading-power-taxonomy.md`（共享 references 目录）
- Genre Profile: `references/genre-profiles.md`
- Context Contract: `references/step-1.5-contract.md`
- Shared References: `references/shared/` 为单一事实源；遇到 `<!-- DEPRECATED:` 标记的文件一律跳过

**执行流程**：

1. **CLI 入口校验**（必做）：
```bash
if [ -z "${SCRIPTS_DIR}" ] || [ ! -d "${SCRIPTS_DIR}" ]; then
  echo "ERROR: 未设置 SCRIPTS_DIR 或缺少目录" >&2
  exit 1
fi
python -X utf8 "${SCRIPTS_DIR}/webnovel.py" --project-root "${PROJECT_ROOT}" where
```

2. **ContextManager 快照优先**：
```bash
python -X utf8 "${SCRIPTS_DIR}/webnovel.py" --project-root "${PROJECT_ROOT}" context -- --chapter {chapter_padded}
```

3. **Context Contract 上下文包**：
```bash
python -X utf8 "${SCRIPTS_DIR}/webnovel.py" --project-root "${PROJECT_ROOT}" extract-context --chapter {chapter_padded} --format json
```
- 必须读取：`writing_guidance.guidance.items`
- 推荐读取：`reader_signal` 与 `genre_profile.reference_hints`
- 条件读取：`rag_assist`（当 `invoked=true` 且 `hits` 非空时，必须提炼成可执行约束）

4. **时间线读取**（必做）：
- 从 `state.json` 或 `大纲/总纲.md` 确定 `{volume_id}`
- 读取 `大纲/第{volume_id}卷-时间线.md`
- 从章纲提取：时间锚点、章内时间跨度、与上章时间差、倒计时状态

**读取优先级与默认值**：

| 字段 | 读取来源 | 缺失时默认值 |
|------|---------|-------------|
| 上章钩子 | `chapter_meta[NNNN].hook` 或 `chapter_reading_power` | `{type: "无", content: "上章无明确钩子", strength: "weak"}` |
| 最近3章模式 | `chapter_meta` 或 `chapter_reading_power` | 空数组，不做重复检查 |
| 上章结束情绪 | `chapter_meta[NNNN].ending.emotion` | "未知"（提示自行判断） |
| 角色动机 | 从大纲+角色状态推断 | **必须推断，无默认值** |
| 题材Profile | `state.json → project.genre` | 默认 "shuangwen" |
| 当前债务 | `index.db → chase_debt` | 0 |

**缺失处理**：
- 若 `chapter_meta` 不存在（如第1章），跳过"接住上章"
- 最近3章数据不完整时，只用现有数据做差异化检查
- 若 `plot_threads.foreshadowing` 缺失或非列表：视为"当前无结构化伏笔数据"，第 7 板块输出空清单并显式标注"数据缺失，需人工补录"

**关键数据来源**：
- `state.json`: 进度、主角状态、strand_tracker、chapter_meta、project.genre、plot_threads.foreshadowing
- `index.db`: 实体/别名/关系/状态变化/override_contracts/chase_debt/chapter_reading_power
- `.webnovel/summaries/ch{NNNN}.md`: 章节摘要（含钩子/结束状态）
- `.webnovel/context_snapshots/`: 上下文快照（优先复用）
- `大纲/` 与 `设定集/`

**输出格式：创作执行包（Step 2A 直连）**

输出必须是单一执行包，包含 3 层：

1. **任务书（8板块）**
- 本章核心任务（目标/阻力/代价、冲突一句话、必须完成、绝对不能、反派层级）
- 接住上章（上章钩子、读者期待、开头建议）
- 出场角色（状态、动机、情绪底色、说话风格、红线）
- 场景与力量约束（地点、可用能力、禁用能力）
- **时间约束**（上章时间锚点、本章时间锚点、允许推进跨度、时间过渡要求、倒计时状态）
- 风格指导（本章类型、参考样本、最近模式、本章建议）
- 连续性与伏笔（时间/位置/情绪连贯；必须处理/可选伏笔）
- 追读力策略（未闭合问题 + 钩子类型/强度、微兑现建议、差异化提示）

2. **Context Contract（内置 Step 1.5）**
- 目标、阻力、代价、本章变化、未闭合问题、核心冲突一句话
- 开头类型、情绪节奏、信息密度
- 是否过渡章（必须按大纲判定，禁止按字数判定）
- 追读力设计（钩子类型/强度、微兑现清单、爽点模式）

3. **Step 2A 直写提示词**
- 章节节拍（开场触发 → 推进/受阻 → 反转/兑现 → 章末钩子）
- 不可变事实清单（大纲事实/设定事实/承接事实）
- 禁止事项（越级能力、无因果跳转、设定冲突、剧情硬拐）
- 终检清单（本章必须满足项 + fail 条件）

**硬要求**：
- 三层信息必须一致；若冲突，以"设定 > 大纲 > 风格偏好"优先。
- 输出内容必须能直接给 Step 2A 开写，不再依赖额外补问。
- 若 `state` 或大纲不可用，立即阻断并返回缺失项。
- 输出必须同时包含：7 板块任务书、Context Contract 全字段、Step 2A 直写提示词。
- 合同与任务书出现冲突时，以"大纲与设定约束更严格者"为准。

**输出**：
- 单一“创作执行包”（任务书 + Context Contract + 直写提示词），供 Step 2A 直接消费，不再拆分独立 Step 1.5。

### Step 2A：正文起草

执行前必须加载：
```bash
cat "${SKILL_ROOT}/../../references/shared/core-constraints.md"
```

硬要求：
- 只输出纯正文到章节正文文件；若详细大纲已有章节名，优先使用 `正文/第{chapter_padded}章-{title_safe}.md`，否则回退为 `正文/第{chapter_padded}章.md`。
- 默认按 2000-2500 字执行；若大纲为关键战斗章/高潮章/卷末章或用户明确指定，则按大纲/用户优先。
- 禁止占位符正文（如 `[TODO]`、`[待补充]`）。
- 保留承接关系：若上章有明确钩子，本章必须回应（可部分兑现）。

中文思维写作约束（硬规则）：
- **禁止"先英后中"**：不得先用英文工程化骨架（如 ABCDE 分段、Summary/Conclusion 框架）组织内容，再翻译成中文。
- **中文叙事单元优先**：以"动作、反应、代价、情绪、场景、关系位移"为基本叙事单元，不使用英文结构标签驱动正文生成。
- **禁止英文结论话术**：正文、审查说明、润色说明、变更摘要、最终报告中不得出现 Overall / PASS / FAIL / Summary / Conclusion 等英文结论标题。
- **英文仅限机器标识**：CLI flag（`--fast`）、checker id（`consistency-checker`）、DB 字段名（`anti_ai_force_check`）、JSON 键名等不可改的接口名保持英文，其余一律使用简体中文。

输出：
- 章节草稿（可进入 Step 2B 或 Step 3）。

### Step 2B：风格适配（`--fast` / `--minimal` 跳过）

执行前加载：
```bash
cat "${SKILL_ROOT}/references/style-adapter.md"
```

硬要求：
- 只做表达层转译，不改剧情事实、事件顺序、角色行为结果、设定规则。
- 对“模板腔、说明腔、机械腔”做定向改写，为 Step 4 留出问题修复空间。

输出：
- 风格化正文（覆盖原章节文件）。

### Step 3：审查（auto 路由，必须由 Task 子代理执行）

执行前加载：
```bash
cat "${SKILL_ROOT}/references/step-3-review-gate.md"
```

调用约束：
- 必须用 `Task` 调用审查 subagent，禁止主流程伪造审查结论。
- 可并行发起审查，统一汇总 `issues/severity/overall_score`。
- 默认使用 `auto` 路由：根据“本章执行合同 + 正文信号 + 大纲标签”动态选择审查器。

核心审查器（始终执行）：
- `consistency-checker`
- `continuity-checker`
- `ooc-checker`

条件审查器（`auto` 命中时执行）：
- `reader-pull-checker`
- `high-point-checker`
- `pacing-checker`

模式说明：
- 标准/`--fast`：核心 3 个 + auto 命中的条件审查器
- `--minimal`：只跑核心 3 个（忽略条件审查器）

审查指标落库（必做）：
```bash
python -X utf8 "${SCRIPTS_DIR}/webnovel.py" --project-root "${PROJECT_ROOT}" index save-review-metrics --data "@${PROJECT_ROOT}/.webnovel/tmp/review_metrics.json"
```

review_metrics 字段约束（当前工作流约定只传以下字段）：
```json
{
  "start_chapter": 100,
  "end_chapter": 100,
  "overall_score": 85.0,
  "dimension_scores": {"爽点密度": 8.5, "设定一致性": 8.0, "节奏控制": 7.8, "人物塑造": 8.2, "连贯性": 9.0, "追读力": 8.7},
  "severity_counts": {"critical": 0, "high": 1, "medium": 2, "low": 0},
  "critical_issues": ["问题描述"],
  "report_file": "审查报告/第100-100章审查报告.md",
  "notes": "单个字符串；selected_checkers / timeline_gate / anti_ai_force_check 等扩展信息压成单行文本写入此字段"
}
```
- `notes` 在当前执行契约中必须是单个字符串，不得传入对象或数组。
- 当前工作流不额外传入其它顶层字段；脚本侧未在此处做新增硬校验。

硬要求：
- `--minimal` 也必须产出 `overall_score`。
- 未落库 `review_metrics` 不得进入 Step 5。

### Step 4：润色（问题修复优先）

执行前必须加载：
```bash
cat "${SKILL_ROOT}/references/polish-guide.md"
cat "${SKILL_ROOT}/references/writing/typesetting.md"
```

执行顺序：
1. 修复 `critical`（必须）
2. 修复 `high`（不能修复则记录 deviation）
3. 处理 `medium/low`（按收益择优）
4. 执行 Anti-AI 与 No-Poison 全文终检（必须输出 `anti_ai_force_check: pass/fail`）

输出：
- 润色后正文（覆盖章节文件）
- 变更摘要（至少含：修复项、保留项、deviation、`anti_ai_force_check`）

### Step 5：Data Agent（状态与索引回写）

> **Role**: 智能数据工程师，负责从章节正文中提取结构化信息并写入数据链。
> **Principle**: AI驱动提取，智能消歧 - 用语义理解替代正则匹配，用置信度控制质量。

**输入参数**：
```json
{
  "chapter": {chapter_num},
  "chapter_file": "正文/第{chapter_padded}章-{title_safe}.md",
  "review_score": {Step 3 overall_score},
  "project_root": "${PROJECT_ROOT}",
  "storage_path": ".webnovel/",
  "state_file": ".webnovel/state.json"
}
```

`chapter_file` 必须传入实际章节文件路径。若详细大纲已有章节名，优先使用带标题文件名；旧的 `正文/第{chapter_padded}章.md` 仍兼容。

**数据写入位置**（所有数据写入 `{project_root}/.webnovel/` 目录）：
- index.db → 实体、别名、状态变化、关系、章节索引 (SQLite)
- state.json → 进度、配置、节奏追踪 + chapter_meta
- vectors.db → RAG 向量 (SQLite)
- summaries/ → 章节摘要文件

**执行流程**：

**Step -1: CLI 入口校验**（必做）：
```bash
if [ -z "${SCRIPTS_DIR}" ] || [ ! -d "${SCRIPTS_DIR}" ]; then
  echo "ERROR: 未设置 SCRIPTS_DIR" >&2
  exit 1
fi
python -X utf8 "${SCRIPTS_DIR}/webnovel.py" --project-root "${PROJECT_ROOT}" preflight
python -X utf8 "${SCRIPTS_DIR}/webnovel.py" --project-root "${PROJECT_ROOT}" where
```

**Step A: 加载上下文**（SQL 查询）：
- 使用 Read 工具读取章节正文
- 使用 Bash 工具从 index.db 查询已有实体：
```bash
python -X utf8 "${SCRIPTS_DIR}/webnovel.py" --project-root "${PROJECT_ROOT}" index get-core-entities
python -X utf8 "${SCRIPTS_DIR}/webnovel.py" --project-root "${PROJECT_ROOT}" index get-aliases --entity "xiaoyan"
python -X utf8 "${SCRIPTS_DIR}/webnovel.py" --project-root "${PROJECT_ROOT}" index recent-appearances --limit 20
python -X utf8 "${SCRIPTS_DIR}/webnovel.py" --project-root "${PROJECT_ROOT}" index get-by-alias --alias "萧炎"
```

**Step B-I**: 实体提取 → 消歧 → 写入 state/index → 摘要 → 场景切片 → RAG 索引 → 风格评估 → 债务利息。详细输出格式、置信度策略、scenes 优先级、观测日志见 `references/data-agent-output.md`。

**Step 5 失败隔离规则**：
- 若 G/H 失败原因是 `--scenes` 缺失、scene 为空、scene JSON 格式错误：只补跑 G/H 子步骤，不回滚或重跑 Step 1-4。
- 若 A-E 失败（state/index/summary 写入失败）：仅重跑 Step 5，不回滚已通过的 Step 1-4。
- 禁止因 RAG/style 子步骤失败而重跑整个写作链。

**执行后检查**（最小白名单）：
- `.webnovel/state.json`
- `.webnovel/index.db`
- `.webnovel/summaries/ch{chapter_padded}.md`
- `.webnovel/observability/data_agent_timing.jsonl`（观测日志）

**债务利息**：
- 默认关闭，仅在用户明确要求或开启追踪时执行（见 `step-5-debt-switch.md`）。

### Step 6：Git 备份（可失败但需说明）

```bash
git add .
git -c i18n.commitEncoding=UTF-8 commit -m "第{chapter_num}章: {title}"
```

规则：
- 提交时机：验证、回写、清理全部完成后最后执行。
- 提交信息默认中文，格式：`第{chapter_num}章: {title}`。
- 若 commit 失败，必须给出失败原因与未提交文件范围。

## 充分性闸门（必须通过）

未满足以下条件前，不得结束流程：

1. 章节正文文件存在且非空：`正文/第{chapter_padded}章-{title_safe}.md` 或 `正文/第{chapter_padded}章.md`
2. Step 3 已产出 `overall_score` 且 `review_metrics` 成功落库
3. Step 4 已处理全部 `critical`，`high` 未修项有 deviation 记录
4. Step 4 的 `anti_ai_force_check=pass`（基于全文检查；fail 时不得进入 Step 5）
5. Step 5 已回写 `state.json`、`index.db`、`summaries/ch{chapter_padded}.md`
6. 若开启性能观测，已读取最新 timing 记录并输出结论

## 验证与交付

执行检查：

```bash
test -f "${PROJECT_ROOT}/.webnovel/state.json"
test -f "${PROJECT_ROOT}/正文/第${chapter_padded}章.md"
test -f "${PROJECT_ROOT}/.webnovel/summaries/ch${chapter_padded}.md"
python -X utf8 "${SCRIPTS_DIR}/webnovel.py" --project-root "${PROJECT_ROOT}" index get-recent-review-metrics --limit 1
tail -n 1 "${PROJECT_ROOT}/.webnovel/observability/data_agent_timing.jsonl" || true
```

成功标准：
- 章节文件、摘要文件、状态文件齐全且内容可读。
- 审查分数可追溯，`overall_score` 与 Step 5 输入一致。
- 润色后未破坏大纲与设定约束。

## 失败处理（最小回滚）

触发条件：
- 章节文件缺失或空文件；
- 审查结果未落库；
- Data Agent 关键产物缺失；
- 润色引入设定冲突。

恢复流程：
1. 仅重跑失败步骤，不回滚已通过步骤。
2. 常见最小修复：
   - 审查缺失：只重跑 Step 3 并落库；
   - 润色失真：恢复 Step 2A 输出并重做 Step 4；
   - 摘要/状态缺失：只重跑 Step 5；
3. 重新执行“验证与交付”全部检查，通过后结束。
