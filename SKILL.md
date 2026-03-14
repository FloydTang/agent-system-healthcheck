---
name: agent-system-healthcheck
description: |
  AI 多代理系统架构体检与优化。对 OpenClaw 或类似多代理系统执行全面健康检查：
  System Prompt 注入量审计、Token 消耗评估、硬编码检测、Cron 冲突扫描、
  Skill 规范合规检查、文件冗余清理。按 P0/P1/P2 分级执行优化并量化前后对比。
  触发场景：系统体检、架构审查、Token 优化、prompt 瘦身、cron 审计、
  大版本升级后健康检查、"帮我看看系统有没有问题"。
---

# Agent System Healthcheck

多代理系统架构体检的标准化四阶段流程：扫描 → 诊断 → 治疗 → 验证。

## 适用场景

- 系统大版本升级后
- Token 消耗异常增长
- 新增/删除代理后的配置一致性检查
- 定期维护（建议每月一次）

## 阶段一：扫描

### 1.1 System Prompt 注入量统计

统计所有被自动注入到 system prompt 的文件：

```bash
# OpenClaw 默认注入文件列表
FILES="AGENTS.md SOUL.md TOOLS.md IDENTITY.md USER.md HEARTBEAT.md MEMORY.md BOOTSTRAP.md"
WORKSPACE="/root/.openclaw/agents/main/workspace"

total=0
for f in $FILES; do
  path="$WORKSPACE/$f"
  if [ -f "$path" ]; then
    chars=$(wc -c < "$path")
    tokens=$((chars / 4))
    echo "$f: ${chars} chars (~${tokens} tok)"
    total=$((total + chars))
  fi
done
echo "--- Total: ${total} chars (~$((total / 4)) tok)"
```

**基准线**：注入总量应控制在 **8,000 chars (~2,000 tok)** 以内。超过则进入诊断。

### 1.2 硬编码检测

扫描工作区内的模型 ID、API key、URL 硬编码：

```bash
# 搜索硬编码模型 ID（排除配置文件本身）
grep -rn "claude-\|gpt-\|kimi-\|minimax/" --include="*.py" --include="*.md" \
  --exclude-dir=".git" --exclude="ROUTING.md" --exclude="TOOLS.md"

# 搜索潜在 API key 泄露
grep -rn "sk-\|key.*=.*['\"]" --include="*.py" --include="*.sh" \
  --exclude-dir=".git" --exclude="*.env"
```

### 1.3 文件冗余扫描

查找备份文件、空文件、重复内容：

```bash
# 备份文件
find . -name "*.bak" -o -name "*.old" -o -name "*~" -o -name "*.orig"

# 空文件（0 字节）
find . -empty -type f -not -path "./.git/*"

# 大文件（>50KB，可能不该在 workspace 里）
find . -size +50k -type f -not -path "./.git/*"
```

### 1.4 Cron 任务快照

获取当前所有 cron 任务，记录时间和类型以便后续冲突检测。详见 `references/cron-audit.md`。

## 阶段二：诊断

### 2.1 Token 消耗评估

对每个注入文件评分：

| 等级 | 标准 | 动作 |
|------|------|------|
| 🟢 精简 | < 500 chars，信息密度高 | 保持 |
| 🟡 偏胖 | 500-2000 chars，有可压缩空间 | 考虑瘦身 |
| 🔴 臃肿 | > 2000 chars，或含详细说明/示例 | 必须瘦身 |

**瘦身策略**：将详细内容移至 `memory/topics/` 或 `references/`，注入文件只保留索引和核心规则。

### 2.2 Cron 冲突检测

检查项（详见 `references/cron-audit.md`）：
- 同一时间点的多个任务（撞车）
- disabled 但未删除的死任务
- systemEvent 类型用于需要可靠执行的任务（应改为 isolated agentTurn）
- 时区不统一

### 2.3 Skill 规范合规

对每个 Skill 检查：
- YAML frontmatter 是否包含 name + description
- 是否有多余文件（README.md、CHANGELOG.md 等）
- references/ 是否被 SKILL.md 正确引用
- 行数是否超过 500 行

### 2.4 配置一致性

- 每个 agent 的 binding 是否与 channel 配置匹配
- allowFrom 是否覆盖所有需要的频道
- 模型 fallback 链是否有效

## 阶段三：治疗

### 3.1 问题分级

| 级别 | 定义 | 示例 |
|------|------|------|
| **P0** | 影响功能或造成资源浪费 | Cron 撞车、注入文件 > 3000 chars、硬编码 API key |
| **P1** | 影响效率或规范性 | 硬编码模型 ID、时区不统一、死任务未清理 |
| **P2** | 改善型优化 | 备份文件残留、Skill 行数偏多 |

### 3.2 执行原则

1. **先备份后修改** — 大改前 git commit 当前状态
2. **逐项执行** — 每个 P0 修复后验证再继续
3. **量化记录** — 每项修改记录 before/after 数据
4. **不碰红线** — 不修改 openclaw.json（除非有临时授权）

### 3.3 常见治疗手段

详见 `references/optimization-playbook.md`。

## 阶段四：验证

### 4.1 前后对比表

体检完成后输出标准对比表：

```markdown
| 指标 | 体检前 | 体检后 | 变化 |
|------|--------|--------|------|
| 注入总量 (chars) | ? | ? | -?% |
| 估算 tokens | ? | ? | -? tok |
| Cron 任务数 | ? | ? | ±? |
| 硬编码残留 | ? | 0 | -? |
| 备份文件 | ? | 0 | -? |
```

### 4.2 零残留确认

```bash
# 确认无硬编码残留
grep -rn "硬编码关键词" --include="*.py" | wc -l  # 应为 0

# 确认无备份文件
find . -name "*.bak" -o -name "*.old" | wc -l  # 应为 0
```

### 4.3 记录到工作日志

将体检结果追加到 `memory/YYYY-MM-DD.md`，包含：
- 发现的问题数量和分级
- 执行的修复
- 前后对比数据
- 遗留待办（如有）
