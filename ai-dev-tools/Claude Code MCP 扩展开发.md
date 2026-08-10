# Claude Code MCP 服务器与扩展开发

> 最后更新: 2026-08-09

## 目录
- [MCP 简介](#mcp-简介)
- [内置 MCP 服务器](#内置-mcp-服务器)
- [配置 MCP 服务器](#配置-mcp-服务器)
- [开发自定义 MCP 服务器](#开发自定义-mcp-服务器)
- [实用工具集成](#实用工具集成)
- [最佳实践](#最佳实践)

---

## MCP 简介

MCP (Model Context Protocol) 是 Anthropic 推出的开放标准，用于连接 AI 模型与外部工具和数据源。

**核心概念：**
- **Server**: 提供工具和资源的进程
- **Client**: Claude Code 等 AI 客户端
- **Tool**: 可执行的操作
- **Resource**: 可访问的数据
- **Prompt**: 可复用的提示模板

---

## 内置 MCP 服务器

### 官方 MCP 服务器

| 服务器 | 用途 | 安装 |
|--------|------|------|
| github | GitHub API 集成 | `npx @modelcontextprotocol/server-github` |
| filesystem | 文件系统访问 | 内置 |
| memory | 向量记忆存储 | 内置 |
| context7 | 文档查询 | `npx -y @upstash/context7-mcp` |
| fetch | Web 请求 | `npx @modelcontextprotocol/server-fetch` |

### 第三方 MCP 服务器

| 服务器 | 用途 |
|--------|------|
| postgres | PostgreSQL 数据库 |
| sqlite | SQLite 数据库 |
| brave-search | 网络搜索 |
| slack | Slack 集成 |
| linear | Linear 项目管理 |
| notion | Notion 集成 |
| figma | Figma 设计工具 |
| jira | Atlassian Jira |

---

## 配置 MCP 服务器

### settings.json 配置

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "ghp_..."
      }
    },
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "postgresql://..."
      }
    }
  }
}
```

### 环境变量配置

```bash
# GitHub MCP
export GITHUB_TOKEN="ghp_..."

# PostgreSQL MCP
export DATABASE_URL="postgresql://user:pass@host:5432/db"

# Brave Search MCP
export BRAVE_API_KEY="..."
```

---

## 开发自定义 MCP 服务器

### Node.js 示例

```javascript
// mcp-server-mytool/index.js
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
} from "@modelcontextprotocol/sdk/types.js";

const server = new Server({
  name: "my-tool-server",
  version: "1.0.0",
}, {
  capabilities: {
    tools: {},
  },
});

// 定义工具
server.setRequestHandler(ListToolsRequestSchema, async () => {
  return {
    tools: [
      {
        name: "get_weather",
        description: "Get current weather for a location",
        inputSchema: {
          type: "object",
          properties: {
            location: {
              type: "string",
              description: "City name or coordinates",
            },
          },
          required: ["location"],
        },
      },
    ],
  };
});

// 处理工具调用
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === "get_weather") {
    const { location } = request.params.arguments;
    // 调用天气 API
    const weather = await fetchWeather(location);
    return {
      content: [
        {
          type: "text",
          text: JSON.stringify(weather),
        },
      ],
    };
  }
  throw new Error(`Unknown tool: ${request.params.name}`);
});

async function fetchWeather(location) {
  // 实现天气查询逻辑
  return { location, temperature: 22, condition: "sunny" };
}

const transport = new StdioServerTransport();
await server.connect(transport);
```

### 配置使用

```json
{
  "mcpServers": {
    "weather": {
      "command": "node",
      "args": ["/path/to/mcp-server-mytool/index.js"]
    }
  }
}
```

### Python 示例

```python
# mcp-server-mytool/main.py
from mcp.server import Server
from mcp.types import Tool, TextContent

app = Server("my-tool-server")

@app.tool()
async def get_weather(location: str) -> list:
    """Get current weather for a location."""
    weather = await fetch_weather(location)
    return [TextContent(type="text", text=str(weather))]

async def fetch_weather(location: str) -> dict:
    # 实现天气查询
    return {"location": location, "temperature": 22}

if __name__ == "__main__":
    import asyncio
    asyncio.run(app.run())
```

---

## 实用工具集成

### GitHub 工作流

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-github"]
    }
  }
}
```

**可用操作：**
- 创建/管理 Issues
- 查看 Pull Requests
- 读取代码
- 创建分支
- 获取仓库信息

### 数据库集成

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "postgresql://user:pass@localhost:5432/mydb"
      }
    },
    "sqlite": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sqlite"],
      "env": {
        "DATABASE_PATH": "/path/to/database.db"
      }
    }
  }
}
```

### 搜索集成

```json
{
  "mcpServers": {
    "brave-search": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-brave-search"],
      "env": {
        "BRAVE_API_KEY": "..."
      }
    },
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    }
  }
}
```

### 项目管理

```json
{
  "mcpServers": {
    "linear": {
      "command": "npx",
      "args": ["-y", "@linear/mcp-server"]
    },
    "notion": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-notion"],
      "env": {
        "NOTION_API_KEY": "..."
      }
    }
  }
}
```

---

## 最佳实践

### 1. 安全配置

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"  // 从环境变量读取
      }
    }
  }
}
```

### 2. 性能优化

```json
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "...",
        "QUERY_TIMEOUT": "5000"  // 查询超时
      }
    }
  }
}
```

### 3. 错误处理

```javascript
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  try {
    // 工具逻辑
    return { content: [...] };
  } catch (error) {
    return {
      content: [
        {
          type: "text",
          text: `Error: ${error.message}`,
        },
      ],
      isError: true,
    };
  }
});
```

### 4. 日志记录

```javascript
const logger = {
  info: (msg) => console.error(`[MCP][INFO] ${msg}`),
  error: (msg) => console.error(`[MCP][ERROR] ${msg}`),
};
```

---

## 常用 MCP 命令

```bash
# 列出已安装的 MCP 服务器
claude mcp list

# 添加 MCP 服务器
claude mcp add github npx @modelcontextprotocol/server-github

# 移除 MCP 服务器
claude mcp remove github

# 测试 MCP 服务器
claude mcp test github
```

---

## 参考资料

- [MCP 官方文档](https://modelcontextprotocol.io)
- [MCP SDK GitHub](https://github.com/modelcontextprotocol)
- [Claude Code MCP 配置](https://docs.anthropic.com/en/docs/claude-code/mcp)

---

## 相关资源

- [Model Context Protocol 规范](https://modelcontextprotocol.io/specification)
- [官方 MCP 服务器列表](https://github.com/modelcontextprotocol/servers)
- [社区 MCP 服务器](https://github.com/modelcontextprotocol/servers)
- [MCP SDK 文档](https://modelcontextprotocol.io/sdk)
