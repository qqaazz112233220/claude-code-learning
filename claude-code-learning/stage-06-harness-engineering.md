# 第六阶段：深度定制与工作流工程（Harness Engineering）

> **目标**：达到能设计完整开发工作流的水平，管理多项目配置
> **预计时间**：4-5 天

---

## 学习清单

### 1. 完整配置分层体系

```
your-project/
├── CLAUDE.md                    # 项目规则（git 管理）
├── .mcp.json                    # 团队共享 MCP（git 管理）
├── .claude/
│   ├── settings.json            # 共享权限/Hooks（git 管理）
│   ├── settings.local.json      # 个人覆盖（gitignore）
│   ├── CLAUDE.md                # 本地私有规则
│   ├── rules/                   # 模块化规则
│   │   ├── testing.md
│   │   ├── code-style.md
│   │   └── security.md
│   ├── skills/                  # 自定义技能
│   │   ├── deploy.skill.md
│   │   └── review.skill.md
│   ├── agents/                  # 自定义子代理
│   │   └── code-reviewer.md
│   └── hooks/                   # 自动化钩子
│       ├── pre-tool-use.sh
│       └── post-tool-use.sh
```

**检验标准**：能从零搭建一个项目的完整配置

### 2. 自定义 Sub-agent

`.claude/agents/code-reviewer.md`：

```markdown
---
name: code-reviewer
model: claude-opus-4-7
tools: [Read, Grep, Glob]
instructions: |
  你是一个严格的代码审查者。审查时关注：
  - 安全漏洞
  - 性能问题
  - 代码规范
  - 测试覆盖
---

```

**检验标准**：子代理能独立完成专项任务

### 3. 团队配置管理

- 将 `.claude/settings.json` 提交到 Git 仓库
- 统一权限白名单（防止误操作）
- 共享 Hooks（lint-on-save、block-secrets）
- 组织级强制策略

**检验标准**：为团队设计一份标准配置文件

### 4. Monorepo 配置策略

```
monorepo/
├── CLAUDE.md                 # 组织级标准（启动时加载）
├── packages/
│   ├── api/
│   │   ├── CLAUDE.md         # API 特定规则
│   │   └── src/
│   └── web/
│       └── CLAUDE.md         # Web 特定规则
└── .claude/
    └── rules/
        ├── testing.md        # 启动时加载
        └── api-routes.md     # path 限定规则
```

**检验标准**：为 monorepo 设计分层配置

### 5. Routines（云端代理）

按计划或事件触发的云端代理：

- 每日自动检查依赖更新
- GitHub PR 创建时自动审查
- 定时运行测试套件
- 告警触发时分析错误日志

**检验标准**：设置一个每日自动检查依赖更新的 Routine

### 6. 会话管理系统

社区实践（claude_bigmode 模式）：

| 技能 | 作用 |
|------|------|
| `/wrap-up` | 结束会话时保存进度、更新 TODO、写入 HANDOFF.md |
| `/resume` | 新会话读取 HANDOFF.md → 验证 TODO → 继续工作 |
| `/eow-review` | 周终评审：差异对比、安全审查 |

文件结构：

```
CLAUDE.md        # 项目规则
TODO.md          # Sprint 跟踪，冷启动指针
DECISIONS.md     # 仅追加的 ADR 日志
HANDOFF.md       # gitignored，/wrap-up 写入，/resume 读取
```

**检验标准**：能中断一个任务并在新会话中无缝继续

### 7. 权限与安全策略

Auto Mode 的硬拒绝规则：

```json
{
  "autoMode": {
    "hardDeny": ["rm -rf /*", "DROP TABLE", "git push --force"]
  }
}
```

**检验标准**：配置适合团队的安全权限模型

---

## 学会的标志

> 你能像 DevOps/SRE 管理基础设施一样管理 Claude Code 配置，团队新人加入后开箱即用。
