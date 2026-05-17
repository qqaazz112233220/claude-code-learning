# 第二阶段：项目管理与 CLAUDE.md

> **目标**：能为项目配置持久化知识，让 Claude 理解项目上下文
> **预计时间**：2-3 天

---

## 学习清单

### 1. CLAUDE.md 的作用

- 每次会话启动时自动加载到系统提示词
- 解决 Claude Code 会话"无状态"问题——跨会话传递知识
- **关键**：`/compact` 后自动从磁盘重新加载（聊天中的指令不会保留）

**检验标准**：能给新人解释为什么需要它

### 2. 三层配置体系

| 层级 | 位置 | 作用范围 | 版本控制 |
|------|------|----------|----------|
| 全局用户级 | `~/.claude/CLAUDE.md` | 所有项目、所有机器 | ❌ |
| 项目根目录 | `./CLAUDE.md` | 本项目、所有成员 | ✅ |
| 本地覆盖 | `.claude/CLAUDE.md` | 本项目、仅自己 | ❌ |

此外还有：
- **父目录**遍历（monorepo 多包场景）
- **子目录**按需加载
- **组织级** `/Library/Application Support/ClaudeCode/CLAUDE.md`

**检验标准**：正确划分三个文件的内容

### 3. 内容设计

#### ✅ 应该包含

| 类别 | 示例 |
|------|------|
| 构建/运行命令 | 开发服务器端口、迁移脚本、种子数据 |
| 代码规范 | "使用 ES modules，不用 CommonJS" |
| 测试规则 | 测试运行器、单测命令、Mock 策略 |
| 架构决策 | "所有 API 返回 { data, error } 格式" |
| 注意事项 | "遗留支付模块使用回调，不要改成 Promise" |

#### ❌ 不应该包含

| 排除项 | 原因 |
|--------|------|
| Claude 能从代码中推断的内容 | 浪费 token |
| 标准语言约定（如 PEP 8） | Claude 已经知道 |
| 完整的 API 文档 | 用链接替代 |
| 频繁变化的信息 | 过时的指令比没有更糟 |

> **黄金法则**：每一行问自己"删除这一行会导致 Claude 犯错吗？"如果不会，就删掉。

**检验标准**：写出 20-30 行的有效 CLAUDE.md

### 4. 模块化 rules

`.claude/rules/` 下拆分规则文件：

```
.claude/rules/
├── testing.md        # 测试相关规则
├── code-style.md     # 代码风格
├── security.md       # 安全规范
├── api-routes.md     # API 约定
└── git-workflow.md   # Git 工作流
```

**检验标准**：为项目创建 3 个以上分类规则文件

### 5. 上下文管理

| 命令 | 用途 | 何时用 |
|------|------|--------|
| `/clear` | 完全清空上下文 | 任务彻底结束、上下文污染 |
| `/compact` | 压缩上下文但保留核心 | 会话变长但任务未完成 |
| `/usage` | 查看 token 消耗分析 | 需要诊断性能时 |

**检验标准**：在长会话中主动管理上下文

### 6. settings.json

层级（从低到高覆盖）：`~/.claude/settings.json` < `.claude/settings.json` < `.claude/settings.local.json`

常见配置项：

```json
{
  "model": "claude-sonnet-4-6",
  "theme": "monokai",
  "permissions": {
    "allow": ["npm install", "git *"],
    "deny": ["rm -rf"]
  }
}
```

**检验标准**：修改默认模型或主题

---

## 学会的标志

> 你能为自己的项目编写 CLAUDE.md，让 Claude 开箱即用理解项目结构、规范、常用命令。
