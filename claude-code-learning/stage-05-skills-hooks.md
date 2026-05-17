# 第五阶段：Skills 与 Hooks 开发

> **目标**：掌握 Claude Code 的扩展体系，能编写自定义技能和自动化钩子
> **预计时间**：4-5 天

---

## 学习清单

### 1. Skills 概念

**Skill** = 一个 SKILL.md 文件，定义可复用的模块化能力。

Skills 和 CLAUDE.md 的区别：

| | CLAUDE.md | SKILL.md |
|---|-----------|----------|
| 加载时机 | 会话启动自动加载 | 斜杠命令触发或自动匹配 |
| 作用 | 全时段项目知识 | 按需的能力模块 |
| 上下文占用 | 始终占用 | 仅触发时加载 |
| 触发方式 | 自动 | `/skillname` 或规则触发 |

**检验标准**：能解释 Skills 和 CLAUDE.md 的区别

### 2. SKILL.md 格式

```markdown
---
name: lint-fix
description: 自动修复项目的 lint 错误
trigger: url(issue) or type(lint-error)
context: fork
---

运行 linter 并自动修复错误：

1. 运行 `npm run lint` 获取错误列表
2. 逐个修复每个错误
3. 重新运行 `npm run lint` 验证
```

**frontmatter 字段**：

| 字段 | 说明 |
|------|------|
| name | 技能名称，对应 `/name` |
| description | 简短描述（始终加载到上下文） |
| trigger | 自动触发规则 |
| context | `fork`=子代理运行（不阻塞主会话）|

**检验标准**：创建一个 Skill 并成功调用

### 3. 编写 Fork Skill

`context: fork` 让 Skill 在独立子代理中运行：

```markdown
---
name: heavy-analysis
description: 大型代码分析（不阻塞主会话）
context: fork
---

执行深度代码分析任务...
```

**检验标准**：写一个耗时的 Skill 不阻塞主会话

### 4. 参数化 Skill

Skill 接受用户传入参数：

```markdown
---
name: deploy
description: 部署到指定环境（staging/production）
---

部署流程：
1. 确认目标环境：{env}
2. 运行部署脚本
3. 验证部署状态
```

**检验标准**：Skill 能处理不同输入

### 5. 插件市场

```bash
/plugin search database
/plugin install @mcp/postgres
/plugin list
```

**检验标准**：安装并试用一个插件

### 6. Hooks 概念

17 种 Hook 事件，核心事件：

| Hook 事件 | 触发时机 | 典型用途 |
|-----------|----------|----------|
| `PreToolUse` | 工具执行前 | 安全检查、拦截危险操作 |
| `PostToolUse` | 工具执行后 | 自动格式化、记录日志 |
| `UserPromptSubmit` | 用户提交消息前 | 注入额外上下文 |
| `Notification` | 异步通知 | 推送告警 |
| `SessionStart` | 会话启动 | 初始化环境 |
| `SessionStop` | 会话结束 | 清理、总结 |
| `PreFileWrite` | 文件写入前 | 检查敏感信息 |

**检验标准**：能说出 5 个以上 Hook 类型

### 7. 编写 Hook

PreToolUse 拦截敏感操作：

```bash
#!/bin/bash
# hooks/pre-tool-use.sh
if [[ "$TOOL" == "Bash" && "$ARGUMENTS" == *"rm -rf"* ]]; then
  echo "危险操作被拦截！"
  exit 2  # 拦截
fi
exit 0  # 放行
```

PostToolUse 自动格式化：

```bash
#!/bin/bash
# hooks/post-tool-use.sh
if [[ "$TOOL" == "FileWrite" || "$TOOL" == "FileEdit" ]]; then
  npx prettier --write "$FILE_PATH" 2>/dev/null
fi
```

**Hook exit code 含义**：

| Code | 含义 |
|------|------|
| 0 | 放行（允许操作继续） |
| 1 | 告警（操作继续但记录警告） |
| 2 | 拦截（阻止操作） |

**检验标准**：写一个能拦截危险命令的 Hook

### 8. Hooks 配置

在 `.claude/settings.json` 中注册：

```json
{
  "hooks": {
    "PreToolUse": "bash .claude/hooks/pre-tool-use.sh",
    "PostToolUse": "bash .claude/hooks/post-tool-use.sh"
  }
}
```

---

## 学会的标志

> 你有一套自己的 Skill 工具箱和 Hook 自动化网络，团队协作时能统一代码质量和安全规范。
