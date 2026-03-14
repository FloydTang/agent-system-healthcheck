# Agent System Healthcheck ⚡🔬

> AI 多代理系统架构体检与优化 — 标准化四阶段流程

## ✨ 特性

- **System Prompt 注入量审计** — 统计每次对话的 token 成本
- **硬编码检测** — 扫描模型 ID、API key 等硬编码
- **Cron 冲突扫描** — 时间撞车、死任务、类型错配检测
- **Skill 规范合规检查** — frontmatter、文件结构、行数审查
- **P0/P1/P2 分级治疗** — 按优先级逐项修复
- **量化前后对比** — 用数据证明优化效果

## 🎯 价值与意义

多代理系统（如 OpenClaw）运行一段时间后，配置膨胀、token 浪费、任务冲突等问题会悄然积累。本 Skill 提供一套**可重复执行**的标准化体检流程，帮助你：

- 降低每次对话的 token 消耗（实测可节省 ~75%）
- 发现并消除配置冲突和冗余
- 保持系统长期健康运行

**实战数据**：在 Mars（半斤九两科技 AI 系统）上首次执行，System Prompt 注入量从 ~26,000 chars 降至 ~6,600 chars，节省 ~4,844 tok/次对话。

## 🚀 快速开始

### 作为 OpenClaw Skill 安装

将 `agent-system-healthcheck/` 目录复制到你的 skills 目录：

```bash
git clone https://github.com/FloydTang/agent-system-healthcheck.git
cp -r agent-system-healthcheck /path/to/your/skills/
```

### 手动执行

即使不使用 OpenClaw，也可以按照 SKILL.md 中的四阶段流程手动执行体检。

## 📂 项目结构

```
agent-system-healthcheck/
├── SKILL.md                              # 核心流程定义
└── references/
    ├── cron-audit.md                     # Cron 任务审计指南
    └── optimization-playbook.md          # 优化手册（含实战案例）
```

## ⚙️ 四阶段流程

| 阶段 | 内容 | 产出 |
|------|------|------|
| 1. 扫描 | 注入量统计、硬编码检测、文件冗余扫描、Cron 快照 | 数据清单 |
| 2. 诊断 | Token 评估、Cron 冲突检测、Skill 合规、配置一致性 | 问题列表 |
| 3. 治疗 | P0/P1/P2 分级执行修复 | 修复记录 |
| 4. 验证 | 前后对比表、零残留确认 | 体检报告 |

## 🤝 贡献

欢迎提交 Issue 和 PR。特别欢迎：
- 其他多代理框架的适配经验
- 新的检查项建议
- 实战体检数据分享

## 📄 许可证

MIT License

## 📞 联系

- **半斤九两科技** — AI + 数字营销
- GitHub: [@FloydTang](https://github.com/FloydTang)

---

**Version**: 1.0  
**Last Updated**: 2026-03-14
