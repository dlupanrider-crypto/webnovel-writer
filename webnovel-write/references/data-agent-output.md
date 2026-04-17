# Data Agent Output Format

## 实体提取输出

```json
{
  "entities_appeared": [
    {"id": "xiaoyan", "type": "角色", "mentions": ["萧炎", "他"], "confidence": 0.95}
  ],
  "entities_new": [
    {"suggested_id": "hongyi_girl", "name": "红衣女子", "type": "角色", "tier": "装饰"}
  ],
  "state_changes": [
    {"entity_id": "xiaoyan", "field": "realm", "old": "斗者", "new": "斗师", "reason": "突破"}
  ],
  "relationships_new": [
    {"from": "xiaoyan", "to": "hongyi_girl", "type": "相识", "description": "初次见面"}
  ],
  "scenes_chunked": 4,
  "uncertain": [
    {"mention": "那位前辈", "candidates": [{"type": "角色", "id": "yaolao"}, {"type": "角色", "id": "elder_zhang"}], "confidence": 0.6}
  ],
  "warnings": []
}
```

## 置信度策略

| 置信度范围 | 处理方式 |
|-----------|---------|
| 0.85-1.0 | 直接匹配，自动写入 |
| 0.70-0.84 | 标记为 uncertain，记录候选列表 |
| <0.70 | 标记为新实体，建议创建 |

## `--scenes` 来源优先级

1. 优先从 `index.db` 的 scenes 记录获取（Step F 写入的结果）
2. 其次按 `start_line` / `end_line` 从正文切片构造
3. 最后允许单场景退化（整章作为一个 scene）

## 观测日志

- `call_trace.jsonl`：外层流程调用链（agent 启动、排队、环境探测等系统开销）
- `data_agent_timing.jsonl`：Data Agent 内部各子步骤耗时
- 当外层总耗时远大于内层 timing 之和时，默认先归因为 agent 启动与环境探测开销

## 性能要求

- 读取 timing 日志最近一条
- 当 `TOTAL > 30000ms` 时，输出最慢 2-3 个环节与原因说明
