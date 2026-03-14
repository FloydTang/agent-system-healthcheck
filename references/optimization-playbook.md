# 优化手册

## System Prompt 瘦身

### 策略：索引化

将详细内容移至 `memory/topics/` 目录，注入文件只保留：
- 核心规则（铁律级别）
- 主题索引（指向详细文件的链接）
- 关键配置（不可省略的参数）

### 实战案例（2026-03-14 Mars 系统）

| 文件 | 瘦身前 | 瘦身后 | 方法 |
|------|--------|--------|------|
| AGENTS.md | 8,344 chars | 1,191 chars | 详细指南移至 memory/topics/agents_guide.md |
| SOUL.md | 7,501 chars | 1,602 chars | 补充细节移至 memory/topics/soul_details.md |
| IDENTITY.md | 636 chars | 0 (删除) | 合并到 SOUL.md |
| BOOTSTRAP.md | ~1,200 chars | 0 (删除) | 内容已过时，直接删除 |
| **总计** | ~26,000 chars | ~6,600 chars | **节省 ~75%** |

### 评估公式

```
节省 tokens = (瘦身前 chars - 瘦身后 chars) / 4
每次对话节省 = 节省 tokens（因为注入是每次对话都发生的）
```

## 硬编码消除

### 策略：集中配置

创建 `config/models.py`（或等效配置文件），定义模型常量：

```python
MODEL_CLASSIFY = "kimi-coding/k2p5"
MODEL_MEMORY_OPS = "kimi-coding/k2p5"
MODEL_DISTILL = "kimi-coding/k2p5"
MODEL_MAIN = "yunwu/claude-opus-4-6"
```

全仓库引用常量，不直接写模型 ID。

### 验证

```bash
# 搜索残留硬编码（排除配置文件本身）
grep -rn "claude-\|gpt-\|kimi-" --include="*.py" \
  --exclude="models.py" --exclude="config.py" | wc -l
# 结果应为 0
```

## Cron 优化

详见 `cron-audit.md`。

## 文件清理

### 安全删除流程

1. `git status` 确认当前状态干净
2. `git add -A && git commit -m "chore: pre-cleanup snapshot"` 备份
3. 执行删除
4. `git add -A && git commit -m "chore: cleanup - 删除 X 个冗余文件"`

### 常见清理目标

- `.bak` / `.old` / `.orig` 文件
- 空目录
- 已废弃的 Skill 版本
- 重复的文档（同一内容存在多处）
