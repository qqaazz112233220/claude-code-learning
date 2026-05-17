# 第四阶段：MCP 生态集成

> **目标**：掌握 Model Context Protocol，将 Claude Code 接入外部系统和数据源
> **预计时间**：3-4 天

---

## 学习清单

### 1. MCP 核心概念

**MCP = Model Context Protocol**，一种开放协议，让 AI 应用连接外部工具和数据源。

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  Claude Code  │────▶│  MCP Server  │────▶│  外部系统    │
│  (MCP 客户端) │◀────│  (工具提供者) │◀────│  DB/API/文件 │
└──────────────┘     └──────────────┘     └──────────────┘
```

**Scope 类型**：

| scope | 配置文件位置 | 作用范围 |
|-------|-------------|----------|
| local | `~/.claude.json` | 本机所有项目 |
| project | `.mcp.json` | 项目共享 |
| user | `~/.claude.json` | 用户全局 |

**检验标准**：用白话讲清楚 MCP 是什么

### 2. 配置 MCP 服务器

`.mcp.json` 示例：

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_TOKEN": "ghp_..." }
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allowed"]
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres", "postgresql://..."]
    }
  }
}
```

**检验标准**：成功配置一个 MCP 服务器并让 Claude 使用

### 3. 官方 MCP 服务器

值得尝试的官方 MCP：

| 服务器 | 用途 |
|--------|------|
| GitHub | 管理 Issue、PR、代码搜索 |
| PostgreSQL | 直接查询数据库 |
| Filesystem | 安全的文件系统访问 |
| Sentry | 查看错误和性能数据 |
| Slack | 查询和发送消息 |
| Puppeteer | 浏览器自动化 |

**检验标准**：让 Claude 通过 MCP 查 GitHub Issue

### 4. 自定义 MCP 服务器

用 Python 编写一个最简单的 MCP 服务器：

```python
from mcp.server import Server, stdio_server

server = Server("weather")

@server.tool()
async def get_weather(city: str) -> str:
    """获取城市天气"""
    return f"{city} 今天晴天，25°C"

if __name__ == "__main__":
    stdio_server.run(server)
```

**检验标准**：写一个返回天气/时事的 MCP 服务器

### 5. MCP Tool Hooks（v2.1.118+）

直接在 Hook 中调用 MCP 工具，无需子进程：

```json
{
  "hooks": {
    "PreToolUse": {
      "type": "mcp_tool",
      "mcpServer": "security-scanner",
      "tool": "check-secrets",
      "args": { "file": "{file_path}" }
    }
  }
}
```

**检验标准**：配置安全扫描 Hook 调 MCP 检查密钥泄露

### 6. 工具搜索（处理大量 MCP）

当 MCP 工具过多时启用自动搜索：

```bash
export ENABLE_TOOL_SEARCH=auto:5
```

**检验标准**：在超过 10 个 MCP 工具时启用搜索

---

## 学会的标志

> 你能根据项目需求接入合适的 MCP 服务器，并在必要时编写自定义 MCP 服务器扩展 Claude 的能力边界。
