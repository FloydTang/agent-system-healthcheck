# Cron 审计指南

## 快照获取

使用 cron tool `action: list` 获取当前所有任务，记录以下字段：

| 字段 | 说明 |
|------|------|
| name | 任务名称 |
| schedule | 触发时间（cron/every/at） |
| payload.kind | systemEvent 或 agentTurn |
| sessionTarget | main 或 isolated |
| enabled | 是否启用 |

## 冲突检测规则

### 时间撞车

同一分钟内不应有 2 个以上任务。解决方案：
- 合并逻辑相近的任务
- 错开 5-15 分钟

### 死任务

`enabled: false` 超过 7 天的任务应删除，不要留着"以防万一"。

### 类型错配

| 场景 | 正确类型 |
|------|----------|
| 需要代理执行复杂逻辑 | isolated agentTurn |
| 仅通知/提醒 | systemEvent |
| 需要工具调用 | isolated agentTurn |

systemEvent 不能调用工具，如果任务需要执行操作（搜索、写文件、发消息），必须用 agentTurn。

### 时区统一

所有 cron 应使用统一时区（建议 Asia/Shanghai 或 UTC），避免混用导致调度混乱。

## 优化后验证

- 任务总数合理（建议 ≤ 10 个）
- 无同分钟撞车
- 无 disabled 死任务
- 类型全部匹配需求
