---
created: 2026-08-10
updated: 2026-08-10
tags:
  - AI开发工具
  - Claude Code
  - 进阶技巧
subject: AI开发工具
type: 进阶技巧
---

# Claude Code 进阶技巧

> 基于官方文档和社区实践的进阶用法

## 一、高级Hooks配置

### 1. PreToolUse Hooks

#### 自动格式化
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash(npm run format)",
        "hooks": ["echo '代码格式化前检查...'"]
      },
      {
        "matcher": "Edit",
        "hooks": [
          "echo '准备编辑: $TOOL_FILE_PATH'",
          "git diff --stat $TOOL_FILE_PATH 2>/dev/null || true"
        ]
      }
    ]
  }
}
```

#### 安全检查
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash(rm -rf)",
        "hooks": ["echo '警告: 检测到危险操作!'", "test -f .dangerous && exit 1"]
      }
    ]
  }
}
```

### 2. PostToolUse Hooks

#### 自动提交
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          "git add $TOOL_FILE_PATH",
          "git diff --cached --stat $TOOL_FILE_PATH"
        ]
      }
    ]
  }
}
```

#### 审计日志
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          "echo \"[$(date)] $TOOL_INPUT\" >> /tmp/claude-audit.log",
          "test $EXIT_CODE -ne 0 && echo '命令失败: $EXIT_CODE'"
        ]
      }
    ]
  }
}
```

## 二、自定义技能开发

### 1. 技能结构
```
~/.claude/skills/my-skill/
├── SKILL.md          # 技能描述
├── trigger.md        # 触发词
└── scripts/          # 执行脚本
    └── main.sh
```

### 2. 技能示例
```markdown
# My Code Review Skill

## Trigger
/codecov

## Description
自动运行代码覆盖率检查并生成报告

## Usage
当用户说"运行覆盖率检查"时触发
```

### 3. 创建技能
```bash
# 创建技能目录
mkdir -p ~/.claude/skills/code-review
cd ~/.claude/skills/code-review

# 创建SKILL.md
cat > SKILL.md << 'EOF'
# 代码覆盖率检查

## 触发词
/codecoverage

## 功能
1. 运行测试
2. 生成覆盖率报告
3. 检查覆盖率阈值
EOF
```

## 三、智能体编排技巧

### 1. Workflow脚本结构
```javascript
export const meta = {
  name: 'code-review-workflow',
  description: '代码审查工作流',
  phases: [
    { title: '分析', detail: '分析代码变更' },
    { title: '审查', detail: '多视角代码审查' },
    { title: '汇总', detail: '生成审查报告' }
  ]
};
```

### 2. 并行审查
```javascript
// 同时运行多个审查
const results = await parallel([
  () => agent("Security review", {phase: 'Review'}),
  () => agent("Performance review", {phase: 'Review'}),
  () => agent("Style review", {phase: 'Review'})
])
```

### 3. 管道处理
```javascript
// 顺序处理
const results = await pipeline(
  files,
  f => agent(`Review ${f}`, {phase: 'Review'}),
  r => agent(`Validate ${r}`, {phase: 'Validate'})
)
```

## 四、MCP服务器开发

### 1. Node.js MCP服务器模板
```javascript
import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';

const server = new McpServer({
  name: 'my-mcp-server',
  version: '1.0.0'
});

// 定义工具
server.tool('get_weather', 'Get weather for a city', {
  city: { type: 'string' }
}, async ({ city }) => {
  return {
    content: [{ type: 'text', text: `Weather in ${city}: 25°C` }]
  };
});

// 启动服务器
const transport = new StdioServerTransport();
await server.connect(transport);
```

### 2. Python MCP服务器模板
```python
from mcp.server import Server
from mcp.types import Tool, TextContent

app = Server("my-mcp-server")

@app.tool()
async def get_weather(city: str) -> list[TextContent]:
    return [TextContent(type="text", text=f"Weather in {city}: 25°C")]

if __name__ == "__main__":
    import asyncio
    asyncio.run(app.run())
```

## 五、性能优化技巧

### 1. 上下文管理
- 使用 `/compact` 压缩对话历史
- 定期清理不需要的上下文
- 使用 `/clear` 开始新对话

### 2. 模型选择
- 简单任务: Haiku（快速、便宜）
- 日常开发: Sonnet（平衡）
- 复杂架构: Opus（最强）

### 3. 配置优化
```json
{
  "agents": {
    "defaults": {
      "model": "claude-sonnet-4-6"
    }
  },
  "permissions": {
    "allow": ["Bash(npm *)", "Bash(git *)"],
    "deny": ["Bash(rm -rf *)"]
  }
}
```

## 六、最佳实践

### 1. 项目级配置
- 在 `.claude/settings.json` 中配置项目特定设置
- 在 `CLAUDE.md` 中编写项目指令
- 使用 `.gitignore` 忽略敏感文件

### 2. 团队协作
- 共享Hooks配置
- 统一技能开发规范
- 定期更新MCP服务器

### 3. 安全建议
- 不要硬编码密钥
- 使用环境变量
- 审查Hooks脚本

---

*整理日期: 2026-08-10*
*来源: 官方文档、社区实践*
