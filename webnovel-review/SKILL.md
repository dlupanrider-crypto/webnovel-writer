---
name: webnovel-review
description: Use when reviewing web novel chapters for quality. Runs six-dimensional checker agents and generates structured quality reports.
version: 5.5.4
metadata:
  openclaw:
    requires:
      env:
        - OPENAI_API_KEY
      bins:
        - python3
    emoji: "🔍"
    homepage: https://github.com/lingfengQAQ/webnovel-writer
---

# Quality Review Skill

## Project Root Guard（必须先确认）

- Claude Code 的“工作区根目录”不一定等于“书项目根目录”。常见结构：工作区为 `D:\wk\xiaoshuo`，书项目为 `D:\wk\xiaoshuo\凡人资本论`。
- 必须先解析真实书项目根（必须包含 `.webnovel/state.json`），后续所有读写路径都以该目录为准。

环境设置（bash 命令执行前）：
```bash
export WORKSPACE_ROOT="${PROJECT_DIR:-$PWD}"
export SKILL_ROOT="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
export SCRIPTS_DIR="${SKILL_ROOT}/../scripts"

if [ ! -d "${SCRIPTS_DIR}" ]; then
  echo "ERROR: 缺少脚本目录: ${SCRIPTS_DIR}" >&2
  exit 1
fi

export PROJECT_ROOT="$(python -X utf8 "${SCRIPTS_DIR}/webnovel.py" --project-root "${WORKSPACE_ROOT}" where)"
```

## 0.5 工作流断点（best-effort，不得阻断主流程）

> 目标：让 `/webnovel-resume` 能基于真实断点恢复。即使 workflow_manager 出错，也**只记录警告**，审查继续。

推荐（bash）：
```bash
python "${SCRIPTS_DIR}/webnovel.py" --project-root "${PROJECT_ROOT}" workflow start-task --command webnovel-review --chapter {end} || true
```

Step 映射（必须与 `workflow_manager.py get_pending_steps("webnovel-review")` 对齐）：
- Step 1：加载参考
- Step 2：加载项目状态
- Step 3：并行调用检查员
- Step 4：生成审查报告
- Step 5：保存审查指标到 index.db
- Step 6：写回审查记录到 state.json
- Step 7：处理关键问题（AskUserQuestion）
- Step 8：收尾（完成任务）

Step 记录模板（bash，失败不阻断）：
```bash
python "${SCRIPTS_DIR}/webnovel.py" --project-root "${PROJECT_ROOT}" workflow start-step --step-id "Step 1" --step-name "加载参考" || true
python "${SCRIPTS_DIR}/webnovel.py" --project-root "${PROJECT_ROOT}" workflow complete-step --step-id "Step 1" --artifacts '{"ok":true}' || true
```

## Review depth

- **Core (default)**: consistency / continuity / ooc / reader-pull
- **Full (关键章/用户要求)**: core + high-point + pacing

## Step 1: 加载参考（按需）

## References（按步骤导航）

- Step 1（必读，硬约束）：[core-constraints.md](../../references/shared/core-constraints.md)
- Step 1（可选，Full 或节奏/爽点相关问题）：[cool-points-guide.md](../../references/shared/cool-points-guide.md)
- Step 1（可选，Full 或节奏/爽点相关问题）：[strand-weave-pattern.md](../../references/shared/strand-weave-pattern.md)
- Step 1（可选，仅在返工建议需要时）：[common-mistakes.md](references/common-mistakes.md)
- Step 1（可选，仅在返工建议需要时）：[pacing-control.md](references/pacing-control.md)

## Reference Loading Levels (strict, lazy)

- L0: 先确定审查深度（Core / Full），再加载参考。
- L1: 只加载 References 区的“必读”条目。
- L2: 仅在问题定位需要时加载 References 区的“可选”条目。

**必读**:
```bash
cat "${SKILL_ROOT}/../../references/shared/core-constraints.md"
```

**建议（Full 或需要时）**:
```bash
cat "${SKILL_ROOT}/../../references/shared/cool-points-guide.md"
cat "${SKILL_ROOT}/../../references/shared/strand-weave-pattern.md"
```

**可选**:
```bash
cat "${SKILL_ROOT}/references/common-mistakes.md"
cat "${SKILL_ROOT}/references/pacing-control.md"
```

## Step 2: 加载项目状态（若存在）

```bash
cat "$PROJECT_ROOT/.webnovel/state.json"
```

## Step 3: 并行调用检查员（Task）

**调用约束**:
- 必须通过 `Task` 工具调用审查 subagent，禁止主流程直接内联审查结论。
- 各 subagent 结果全部返回后再生成总评与优先级。
- 所有检查器输出遵循 `references/checker-output-schema.md` 统一 JSON Schema。

**Core**:
- `consistency-checker`（设定一致性）
- `continuity-checker`（叙事连贯性）
- `ooc-checker`（人物OOC）
- `reader-pull-checker`（追读力）

**Full 追加**:
- `high-point-checker`（爽点密度）
- `pacing-checker`（Strand Weave 节奏）

### 3.1 consistency-checker（设定一致性检查器）

> **职责**: 设定守卫者，执行第二防幻觉定律（设定即物理）。

**检查范围**：
- **第一层: 战力一致性** - 主角境界/等级与 state.json 匹配；使用能力在境界限制内；力量提升遵循既定规则
- **第二层: 设定规则一致性** - 世界观规则不被违反；力量体系逻辑一致；设定集定义不被推翻
- **第三层: 逻辑一致性** - 事件因果关系合理；时间线无矛盾；角色行为与已知设定一致

**危险信号**：
- `POWER_CONFLICT`: 主角使用超出当前境界的能力
- `RULE_VIOLATION`: 违反已建立的世界观规则
- `LOGIC_BREAK`: 事件发展缺乏因果支撑

### 3.2 continuity-checker（叙事连贯性检查器）

> **职责**: 叙事流守卫者，确保场景过渡顺畅、情节线连贯、逻辑一致。

**四层连贯性检查**：
- **第一层: 场景转换流畅度** - 检查场景间过渡是否自然，时间/空间标记是否清晰
  - 评级: A（自然过渡）/ B（略显生硬）/ C（缺少过渡）/ F（完全断裂）
- **第二层: 情节线连贯** - 追踪活跃情节线（主线/支线），检查引入但未解决、解决但无铺垫、中途遗忘的情节线
- **第三层: 伏笔管理** - 检查伏笔设置与回收的合理性
- **第四层: 逻辑流** - 检查事件发展的因果逻辑是否连贯

### 3.3 ooc-checker（人物OOC检查器）

> **职责**: 角色完整性守卫者，防止 OOC（Out-Of-Character）违规。

**执行流程**：
1. **加载角色档案** - 从 `设定集/角色卡/` 读取角色性格、说话风格、价值观、行为倾向
2. **行为采样** - 提取章节中角色的动作和对话
3. **OOC 检测（三级判定）**：
   - **一级: 轻微偏离** - 角色行为略有不同，但有合理的世界观内解释（如触及底线时的情绪变化）
   - **二级: 中度失真** - 角色行为不一致，缺乏充分的铺垫或解释
   - **三级: 严重OOC** - 角色行为与设定完全矛盾，无任何合理解释

### 3.4 reader-pull-checker（追读力检查器）

> **职责**: 审查"读者为什么会点下一章"，执行 Hard/Soft 约束分层。

**核心参考**：
- `references/reading-power-taxonomy.md`（分类法）
- `references/genre-profiles.md`（题材画像）
- `index.db → chapter_reading_power`（章节追读力数据）
- `state.json → chapter_meta`（上章钩子）

**约束分层**：
- **硬约束**（违反 = 必须修复，不可申诉）：
  - HARD-001: 可读性底线（读者无法理解"发生了什么"）- critical
  - HARD-002: 承诺违背（上章承诺本章无回应）- critical
  - HARD-003: 节奏灾难（连续N章无推进）- critical
  - HARD-004: 冲突真空（整章无问题/目标/代价）- high
- **软约束**（可申诉，需说明理由）：
  - SOFT_HOOK_STRENGTH: 钩子强度建议
  - SOFT_PATTERN_REPEAT: 模式重复风险
  - SOFT_MICROPAYOFF: 微兑现密度建议

**输出指标**：
- `hook_present`: 是否有章末钩子
- `hook_type`: 钩子类型（渴望钩/危机钩/悬念钩等）
- `hook_strength`: 钩子强度（weak/medium/strong）
- `prev_hook_fulfilled`: 上章钩子是否兑现
- `micropayoffs`: 本章微兑现清单
- `pattern_repeat_risk`: 模式重复风险
- `debt_balance`: 追读力债务平衡值

### 3.5 high-point-checker（爽点密度检查器）

> **职责**: 读者满足感机制的质量保障专家（爽点设计）。

**核心参考**：
- `references/reading-power-taxonomy.md`（分类法）
- `references/genre-profiles.md`（题材画像）

**8 种标准爽点执行模式**：

| 模式 | 特征 | 最低要求 |
|------|------|----------|
| **装逼打脸** | 嘲讽/废物 → 反转/震惊 | 铺垫 + 反转 + 反应 |
| **扮猪吃虎** | 示弱/隐藏 → 碾压 | 隐藏 + 轻视 + 碾压 |
| **越级反杀** | 实力差距 → 以弱胜强 | 展示差距 + 策略/爆发 + 反转 |
| **打脸权威** | 权威/前辈 → 主角挑战成功 | 建立权威 + 挑战 + 成功 |
| **反派翻车** | 反派得意 → 计划失败 | 反派铺垫 + 主角反制 + 翻车 |
| **甜蜜超预期** | 期待/心动 → 超预期 | 期待 + 超越 + 情绪 |
| **迪化误解** | 主角随意 → 配角脑补 → 读者优越 | 随意行为 + 信息差 + 误解 + 优越 |
| **身份掉马** | 身份伪装 → 揭露 → 震惊 | 隐藏 + 触发 + 揭露 + 群体反应 |

**质量评估**：
- A级：结构完整，情绪层次丰富，读者满足感强
- B级：结构基本完整，效果一般
- C级：结构缺失，爽点不清晰

### 3.6 pacing-checker（节奏检查器）

> **职责**: 节奏分析师，执行 Strand Weave 平衡检查，防止读者疲劳。

**情节线分类**：

| Strand | 指标 | 示例 |
|--------|------|------|
| **Quest** (主线) | 战斗/任务/探索/升级 | 参加宗门大比、探索秘境、击败反派 |
| **Fire** (感情线) | 情感关系/暧昧/友情/羁绊 | 感情发展、师徒情深、兄弟义气 |
| **Constellation** (世界观线) | 势力关系/阵营/社交网络 | 新势力登场、修仙界格局、宗门政治 |

**分类规则**：
- 一章可以有多条情节线的**底色**，但只有**一条主导**
- 主导 = 占据章节内容 ≥ 60%

**平衡检查（Strand Weave 违规）**：
- 从 `state.json` 加载 `strand_tracker` 历史
- 检查连续同类型情节线是否过多（读者疲劳风险）
- 检查各情节线间隔是否合理
- 输出节奏建议（如"建议下一章切换 Fire 线"）

## Step 4: 生成审查报告

保存到：`审查报告/第{start}-{end}章审查报告.md`

**报告结构（精简版）**:
```markdown
# 第 {start}-{end} 章质量审查报告

## 综合评分
- 爽点密度 / 设定一致性 / 节奏控制 / 人物塑造 / 连贯性 / 追读力
- 总评与等级

## 修改优先级
- 🔴 高优先级（必须修改）
- 🟠 中优先级（建议修改）
- 🟡 低优先级（可选优化）

## 改进建议
- 可执行的修复建议
```

**审查指标 JSON（用于趋势统计）**:
```json
{
  "start_chapter": {start},
  "end_chapter": {end},
  "overall_score": 48,
  "dimension_scores": {
    "爽点密度": 8,
    "设定一致性": 7,
    "节奏控制": 7,
    "人物塑造": 8,
    "连贯性": 9,
    "追读力": 9
  },
  "severity_counts": {"critical": 1, "high": 2, "medium": 3, "low": 1},
  "critical_issues": ["设定自相矛盾"],
  "report_file": "审查报告/第{start}-{end}章审查报告.md",
  "notes": ""
}
```

注意：此处只生成审查指标 JSON；落库见 Step 5。

## Step 5: 保存审查指标到 index.db（必做）

```bash
python "${SCRIPTS_DIR}/webnovel.py" --project-root "${PROJECT_ROOT}" index save-review-metrics --data '@review_metrics.json'
```

## Step 6: 写回审查记录到 state.json（必做）

将审查报告记录写回 `state.json.review_checkpoints`，用于后续追踪与回溯（依赖 `update_state.py --add-review`）：
```bash
python "${SCRIPTS_DIR}/webnovel.py" --project-root "${PROJECT_ROOT}" update-state -- --add-review "{start}-{end}" "审查报告/第{start}-{end}章审查报告.md"
```

## Step 7: 处理关键问题

如发现 critical 问题（`severity_counts.critical > 0` 或 `critical_issues` 非空），**必须使用 AskUserQuestion** 询问用户：
- A) 立即修复（推荐）
- B) 仅保存报告，稍后处理

若用户选择 A：
- 输出“返工清单”（逐条 critical 问题 → 定位 → 最小修复动作 → 注意事项）
- 如用户明确授权可直接修改正文文件，则用 `Edit` 对对应章节文件做最小修复，并建议重新运行一次 `/webnovel-review` 验证

若用户选择 B：
- 不做正文修改，仅保留审查报告与指标记录，结束本次审查

## Step 8: 收尾（完成任务）

```bash
python "${SCRIPTS_DIR}/webnovel.py" --project-root "${PROJECT_ROOT}" workflow start-step --step-id "Step 8" --step-name "收尾" || true
python "${SCRIPTS_DIR}/webnovel.py" --project-root "${PROJECT_ROOT}" workflow complete-step --step-id "Step 8" --artifacts '{"ok":true}' || true
python "${SCRIPTS_DIR}/webnovel.py" --project-root "${PROJECT_ROOT}" workflow complete-task --artifacts '{"ok":true}' || true
```
