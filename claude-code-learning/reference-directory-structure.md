# Claude Code 目录文件结构详解（参考文档）

---

## 一、两套配置体系

| 层级 | 位置 | 作用 |
|------|------|------|
| **用户级（全局）** | `~/.claude/` | 所有项目生效 |
| **项目级（本地）** | `your-project/.claude/` + `CLAUDE.md` | 仅当前项目生效，覆盖全局 |

---

## 二、用户级目录 `~/.claude/`

### 位置
- **Windows**：`C:\Users\<你>\.claude\`
- **Mac/Linux**：`~/.claude/`

### 完整结构

```
~/.claude/
├── settings.json              # 全局设置（主题、默认模型等）
├── CLAUDE.md                  # 全局规则（所有项目生效）
├── keybindings.json           # 自定义快捷键
├── scheduled_tasks.json       # 定时任务（/loop 持久化）
└── projects/                  # 各项目持久化数据
    └── <project-hash>/
        ├── memory/            # 持久化记忆
        │   ├── MEMORY.md
        │   ├── user.md
        │   └── feedback.md
        ├── tasks/             # 任务状态
        └── handoffs/          # HANDOFF.md 交班记录
```

---

### 逐个说明

#### `~/.claude/settings.json`

**是什么**：全局配置文件，控制 Claude Code 的默认行为。

**可配置项**：
```json
{
  "model": "claude-sonnet-4-6",
  "theme": "dark",
  "permissions": { "allow": [], "deny": [] }
}
```

**大白话**：类似 VS Code 的 settings.json——设定默认模型、主题、权限。

**生效范围**：所有项目，但会被项目级同名文件覆盖。

---

#### `~/.claude/CLAUDE.md`

**是什么**：全局规则文件。所有项目中 Claude 都会遵守的规矩。

**例子**：
```markdown
- 总是用中文回复
- 代码注释写英文
- 不要创建 README.md 除非我明确要求
```

**大白话**："出厂设置"——进哪个项目都先读一遍这里的规则。

---

#### `~/.claude/keybindings.json`

**是什么**：自定义键盘快捷键。

**例子**：`{ "key": "ctrl+s", "command": "..." }`

**大白话**：默认快捷键不顺手？自己改。

---

#### `~/.claude/scheduled_tasks.json`

**是什么**：通过 `/loop` 创建的持久化定时任务。

**大白话**：设了每天早上 9 点检查依赖？这个文件存着任务，重启还在。

---

#### `~/.claude/projects/<project-hash>/`

**是什么**：每个项目独立的跨会话持久化数据。

| 子目录 | 用途 |
|--------|------|
| `memory/` | 持久化记忆（你的偏好、角色、反馈） |
| `tasks/` | 任务状态 |
| `handoffs/` | HANDOFF.md，跨会话交班 |

**大白话**：这个项目里教过 Claude 的事、做了一半的任务，下次开新会话还在。

---

## 三、项目级目录

### 完整结构

```
your-project/
├── CLAUDE.md                        # 项目规则（git 管理）
├── .mcp.json                        # MCP 服务器配置（git 管理）
└── .claude/
    ├── settings.json                # 项目共享设置（git 管理）
    ├── settings.local.json          # 个人覆盖（gitignore）
    ├── CLAUDE.md                    # 本地私有规则（gitignore）
    ├── rules/                       # 模块化规则
    │   ├── testing.md
    │   ├── code-style.md
    │   └── security.md
    ├── skills/                      # 自定义技能
    │   ├── deploy.skill.md
    │   └── review.skill.md
    ├── agents/                      # 自定义子代理
    │   └── code-reviewer.md
    └── hooks/                       # 自动化脚本
        ├── pre-tool-use.sh
        └── post-tool-use.sh
```

---

### `CLAUDE.md`（根目录）

**是什么**：**最重要的文件**。每次会话启动时自动加载到 Claude 系统提示词。

**底层**：启动 `claude` 时文件内容拼到模型提示词开头，Claude 从一开始就懂项目规矩。

**应该写什么**：
- 构建/运行命令（`npm run dev`、`make test`）
- 代码规范（"用 ES modules，不用 CommonJS"）
- 架构约定（"API 统一返回 `{ data, error }`"）
- 注意事项（"支付模块别动，遗留系统"）

**不应该写什么**：
- Claude 能从代码看出来的 → 浪费 token
- 标准语言规范（PEP 8 等）→ Claude 早会了
- 大段 API 文档 → 给链接就行
- 频繁变化的信息 → 过时指令比没有更糟

**黄金法则**：每行问自己"删掉这行 Claude 会犯错吗？"不会就删。

---

### `.mcp.json`

**是什么**：定义 Claude 能连接哪些外部工具的配置文件。

**例子**：
```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "."]
    }
  }
}
```

**大白话**："Claude 可以用这些外挂"——连数据库、查文件、调 API。

---

### `.claude/settings.json`

**是什么**：项目共享设置，提交到 git 让团队共用。

**例子**：
```json
{
  "permissions": {
    "allow": ["npm install", "git add ."],
    "deny": ["rm -rf"]
  }
}
```

**大白话**：团队统一的白名单/黑名单。

---

### `.claude/settings.local.json`

**是什么**：个人覆盖，**不提交 git**（在 .gitignore 中）。

**覆盖优先级**（从低到高）：
```
~/.claude/settings.json < .claude/settings.json < .claude/settings.local.json
```

**大白话**：你想用亮色主题但团队用暗色？自己在本地改，不影响别人。

---

### `.claude/CLAUDE.md`

**是什么**：本地私有规则，**不提交 git**。

**和根目录 CLAUDE.md 的关系**：
```
./CLAUDE.md            → 团队共享，提交 git
.claude/CLAUDE.md      → 仅自己，不提交
```

---

### `.claude/rules/`

**是什么**：把 CLAUDE.md 拆成按主题分类的小文件。

**文件示例**：
```
.claude/rules/
├── testing.md        # 测试规范
├── code-style.md     # 代码风格
├── security.md       # 安全规范
├── api-routes.md     # API 约定
└── git-workflow.md   # Git 工作流
```

**为什么拆**：CLAUDE.md 太浪费 token，按主题分文件按需加载。

---

### `.claude/skills/`

**是什么**：可复用的模块化能力，按需触发。

**和 CLAUDE.md 的区别**：
| 对比 | CLAUDE.md | SKILL.md |
|------|-----------|----------|
| 加载时机 | 会话启动自动加载 | 斜杠命令触发或自动匹配 |
| 作用 | 持续的项目知识 | 按需使用 |
| 上下文占用 | 始终占用 | 仅触发时加载 |

**例子**：`deploy.skill.md` → `/deploy` 一键部署

---

### `.claude/agents/`

**是什么**：专门的 AI 子代理，有独立角色和工具集。

**例子** `code-reviewer.md`：
```markdown
---
name: code-reviewer
model: claude-opus-4-7
tools: [Read, Grep, Glob]
instructions: |
  严格代码审查，关注安全、性能、规范、测试覆盖。
---
```

**大白话**：好几个"实习生"——一个专门写测试，一个专门做安全。

---

### `.claude/hooks/`

**是什么**：特定事件自动触发的 shell 脚本。

**事件**：`pre-tool-use.sh` / `post-tool-use.sh` / `pre-commit.sh` / `post-commit.sh`

**例子**：
```bash
# pre-tool-use.sh — Claude 改文件前自动格式化
if [ "$TOOL" = "Edit" ]; then npm run format; fi
```

**大白话**：类似 Git Hooks——"当 Claude 做 X 时，自动跑 Y"。

---

## 四、配置优先级（从低到高）

### settings.json
```
1. ~/.claude/settings.json           → 全局默认
2. .claude/settings.json             → 项目共享
3. .claude/settings.local.json       → 个人覆盖（最高）
```

### CLAUDE.md
```
1. ~/.claude/CLAUDE.md               → 全局规则
2. ./CLAUDE.md                       → 项目规则（覆盖全局）
3. .claude/CLAUDE.md                 → 私有规则（不提交）
4. .claude/rules/*.md                → 模块化规则（按需加载）
```

---

## 五、一句话速查

| 文件/目录 | 一句话 |
|-----------|--------|
| `~/.claude/settings.json` | 你的全局偏好 |
| `~/.claude/CLAUDE.md` | 所有项目都守的规矩 |
| `~/.claude/projects/<hash>/memory/` | 项目级跨会话记忆 |
| `./CLAUDE.md` | 项目规矩，团队共享 |
| `.claude/settings.json` | 项目设置，团队共享 |
| `.claude/settings.local.json` | 个人覆盖，不提交 |
| `.claude/CLAUDE.md` | 私有规则，不提交 |
| `.claude/rules/` | 拆成小文件的专项规则 |
| `.claude/skills/` | 可复用的技能包 |
| `.claude/agents/` | 专门的子代理 |
| `.claude/hooks/` | 自动触发脚本 |
| `.mcp.json` | 外挂工具配置 |
