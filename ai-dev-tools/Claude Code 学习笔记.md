# Claude Code 学习笔记

> 最后更新: 2026-08-09

## 目录
- [概述](#概述)
- [安装与配置](#安装与配置)
- [核心命令](#核心命令)
- [技能系统 (Skills)](#技能系统-skills)
- [智能体 (Agents)](#智能体-agents)
- [Hooks 系统](#hooks-系统)
- [设置与配置](#设置与配置)
- [最佳实践](#最佳实践)
- [进阶用法](#进阶用法)

---

## 概述

Claude Code 是 Anthropic 推出的 AI 编程助手 CLI 工具，集成在终端中，能够：

- 理解和修改代码库
- 执行 shell 命令
- 创建和运行测试
- 管理 git 操作
- 调用外部工具 (MCP servers)
- 使用技能 (Skills) 和智能体 (Agents) 完成复杂任务

**核心特点：**
- 基于 Claude 模型 (Sonnet/Opus/Haiku)
- 支持 1M+ token 上下文窗口
- 可扩展的技能系统
- 可配置的 Hooks
- 多智能体协作

---

## 安装与配置

### 安装方式

```bash
# npm 安装
npm install -g @anthropic-ai/claude-code

# 或 bun 安装
bun install -g @anthropic-ai/claude-code

# 或直接使用
npx @anthropic-ai/claude-code
```

### 身份验证

```bash
# 首次运行会自动提示登录
claude

# 或使用 API key
export ANTHROPIC_API_KEY="sk-ant-..."
```

### 模型选择

```bash
# 切换模型
/model

# 常用模型
- claude-sonnet-4-6      # 主要开发工作
- claude-opus-4-5        # 复杂架构决策
- claude-haiku-4-5-20251001  # 轻量任务
```

### 配置文件位置

```bash
# 用户级配置
~/.claude/settings.json
~/.claude/settings.local.json  # 本地覆盖
~/.claude.json  # 权限配置

# 项目级配置
.claude/settings.json
CLAUDE.md  # 项目指令
```

### settings.json 配置项

```json
{
  "permissions": {
    "allow": ["Bash(npm *)", "Bash(git *)"],
    "deny": ["Bash(rm -rf *)"],
    "prompts": {
      "Bash": "ask"
    }
  },
  "commands": {
    "custom": [
      {
        "name": "/build",
        "prompt": "Run the build command for this project",
        "command": "npm run build"
      }
    ]
  },
  "hooks": {
    "PreToolUse": ["echo 'Before tool use'"],
    "PostToolUse": ["echo 'After tool use'"]
  },
  "agents": {
    "defaults": {
      "model": "claude-sonnet-4-6"
    }
  }
}
```

---

## 核心命令

### 基本命令

| 命令 | 说明 |
|------|------|
| `/model` | 切换 AI 模型 |
| `/clear` | 清除对话历史 |
| `/help` | 显示帮助 |
| `/quit` 或 `/exit` | 退出 Claude Code |
| `/compact` | 压缩对话历史 |
| `/cost` | 查看当前会话成本 |
| `/tokens` | 查看 token 使用情况 |

### 快捷操作

- `Ctrl+O` - 显示/隐藏思考过程
- `Alt+T` (macOS) / `Ctrl+O` (Windows) - 切换扩展思考
- `Tab` - 自动补全命令
- `Ctrl+C` - 取消当前操作

### 权限管理

```bash
# 查看当前权限设置
/permissions

# 允许命令
! npm install  # 运行 npm install
```

---

## 技能系统 (Skills)

技能是预定义的指令集合，用于特定任务类型。

### 内置技能类型

| 技能类别 | 示例 |
|---------|------|
| 学术研究 | academic-paper, deep-research |
| 代码质量 | code-review, security-review |
| 文档 | doc-updater, documentation-lookup |
| 内容创作 | article-writing, content-engine |
| 测试 | tdd, e2e |
| 部署 | deployment-patterns, docker-patterns |

### 创建自定义技能

在 `~/.claude/skills/` 或项目 `.claude/skills/` 目录下创建：

```markdown
---
name: my-skill
description: 描述这个技能的用途
trigger: 触发关键词
---

# 技能指令

详细的执行步骤和检查清单...
```

### 常用技能示例

```markdown
## code-review 技能
- 检查代码质量
- 验证测试覆盖
- 安全审计
- 性能检查

## tdd 技能
- 先写测试 (RED)
- 实现功能 (GREEN)
- 重构优化 (IMPROVE)
```

---

## 智能体 (Agents)

智能体是专门化的子代理，用于并行处理复杂任务。

### 可用智能体类型

| 智能体 | 用途 |
|--------|------|
| planner | 实现规划 |
| architect | 系统架构设计 |
| tdd-guide | TDD 测试驱动 |
| code-reviewer | 代码审查 |
| security-reviewer | 安全分析 |
| build-error-resolver | 构建错误修复 |
| e2e-runner | E2E 测试 |
| refactor-cleaner | 代码清理 |
| doc-updater | 文档更新 |
| Explore | 代码库搜索 |

### 智能体使用示例

```bash
# 自动触发 (无需用户提示)
- 复杂功能请求 → 使用 planner
- 代码编写后 → 使用 code-reviewer
- 架构决策 → 使用 architect
```

### 并行执行

```javascript
// 多个智能体并行运行
Agent 1: Security analysis of auth module
Agent 2: Performance review of cache system
Agent 3: Type checking of utilities
```

---

## Hooks 系统

Hooks 是自动执行的 shell 命令，在特定事件触发时运行。

### Hook 类型

| Hook 类型 | 触发时机 |
|-----------|---------|
| PreToolUse | 工具执行前 |
| PostToolUse | 工具执行后 |
| Stop | 会话结束时 |

### 配置 Hooks

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": ["echo 'Pre-check: $TOOL_NAME'"]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Bash",
        "hooks": ["echo 'Post-check: exit code $EXIT_CODE'"]
      }
    ]
  }
}
```

### 常用 Hook 场景

- **PreToolUse**: 验证参数、记录日志
- **PostToolUse**: 自动格式化、运行 linter
- **Stop**: 最终验证、清理临时文件

---

## 设置与配置

### 权限系统

```json
{
  "permissions": {
    "defaults": "ask",
    "allow": [
      "Read",
      "Glob",
      "Grep",
      "Bash(npm *)",
      "Bash(git *)",
      "WebSearch"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Bash(curl * | sh)",
      "Bash(sudo *)"
    ]
  }
}
```

### CLAUDE.md 项目指令

在项目根目录创建 `CLAUDE.md` 文件，定义项目特定规则：

```markdown
# CLAUDE.md

## 项目结构
- src/: 源代码
- tests/: 测试文件
- docs/: 文档

## 编码规范
- 使用 TypeScript
- 遵循 ESLint 配置
- 测试覆盖率 ≥ 80%

## 常用命令
- npm run dev
- npm run test
- npm run build
```

### 记忆系统

存储用户偏好和项目信息：

```markdown
# 记忆文件位置
~/.claude/projects/<project-path>/memory/

# 记忆类型
- user.md: 用户偏好
- feedback.md: 行为反馈
- project.md: 项目信息
- reference.md: 外部资源指针
```

---

## 最佳实践

### 1. 思考后再编码

```
✓ 明确需求，提出假设
✓ 分析多种方案
✓ 确认理解后再实现
✗ 盲目假设，不确认
✗ 隐藏疑惑
```

### 2. 保持简洁

- 最小代码解决特定问题
- 不添加未请求的功能
- 避免过度抽象
- 如果 200 行可以变成 50 行，请重写

### 3. 外科手术式变更

- 只修改必要的代码
- 不改进相邻代码
- 不重构未损坏的代码
- 匹配现有风格

### 4. 目标驱动执行

```
"添加验证" → 为无效输入编写测试 → 使测试通过
"修复 bug" → 编写复现测试 → 修复实现
"重构 X" → 确保前后测试都通过
```

---

## 进阶用法

### 多智能体工作流

```javascript
// 并行搜索
const results = await parallel([
  () => agent("Search for auth patterns"),
  () => agent("Search for error handling"),
  () => agent("Search for API endpoints")
]);

// 管道处理
const processed = await pipeline(
  files,
  f => agent("Review security", {file: f}),
  r => agent("Check completeness", {review: r})
);
```

### 工作流脚本

```javascript
export const meta = {
  name: 'my-workflow',
  description: 'Description of the workflow',
  phases: [
    { title: 'Research', detail: 'Search for patterns' },
    { title: 'Implement', detail: 'Write code' },
    { title: 'Review', detail: 'Code review' }
  ]
};

// 使用 agent() 和 pipeline() 编排任务
```

### MCP 服务器配置

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-github"]
    },
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    }
  }
}
```

### 模型路由策略

| 任务类型 | 推荐模型 | 原因 |
|---------|---------|------|
| 轻量代码生成 | Haiku | 快速、经济 |
| 主要开发 | Sonnet | 平衡性能与成本 |
| 复杂架构 | Opus | 深度推理 |
| 快速审查 | Haiku | 高效反馈 |

---

## 相关资源

- [官方文档](https://docs.anthropic.com/en/docs/claude-code)
- [GitHub 仓库](https://github.com/anthropics/claude-code)
- [技能市场](https://github.com/anthropics/skills)
- [MCP 规范](https://modelcontextprotocol.io)

---

## 参考资料

- [Anthropic Claude Code 官方文档](https://docs.anthropic.com/en/docs/claude-code/overview)
- [Anthropic Claude Code GitHub](https://github.com/anthropics/claude-code)
- [Model Context Protocol (MCP)](https://modelcontextprotocol.io)
- [Claude Code Skills Documentation](https://docs.anthropic.com/en/docs/claude-code/skills)
