# OpenAI Codex CLI 学习笔记

> 最后更新: 2026-08-09

## 目录
- [概述](#概述)
- [安装与配置](#安装与配置)
- [核心命令](#核心命令)
- [使用模式](#使用模式)
- [配置选项](#配置选项)
- [最佳实践](#最佳实践)
- [与 Claude Code 对比](#与-claude-code-对比)

---

## 概述

OpenAI Codex CLI 是 OpenAI 推出的命令行编程助手，基于 GPT-4/Codex 模型，能够在终端中理解和修改代码。

**核心特点：**
- 基于 OpenAI API (GPT-4/Codex)
- 支持交互式对话
- 可执行 shell 命令
- 理解代码库上下文
- 支持多种编程语言

---

## 安装与配置

### 安装方式

```bash
# npm 安装
npm install -g @openai/codex

# 或使用 pnpm
pnpm add -g @openai/codex
```

### 身份验证

```bash
# 设置 API key
export OPENAI_API_KEY="sk-..."

# 或使用配置文件
codex auth login
```

### 初始配置

```bash
# 初始化配置
codex init

# 查看当前配置
codex config
```

---

## 核心命令

### 基本命令

| 命令 | 说明 |
|------|------|
| `codex` | 启动交互式会话 |
| `codex "prompt"` | 直接执行提示 |
| `codex edit <file>` | 编辑指定文件 |
| `codex explain <file>` | 解释代码 |
| `codex fix` | 修复错误 |
| `codex test` | 运行测试 |
| `codex --help` | 显示帮助 |

### 模式切换

```bash
# 全自动模式 (全自动执行)
codex -y "Fix the bug in auth.js"

# 对话模式 (交互式)
codex "Help me refactor this code"

# 审核模式 (只读分析)
codex --auditor "Review this PR"

# 仅建议模式 (不执行)
codex --suggestions "How to improve this function?"
```

### 常用选项

```bash
# 指定模型
codex --model gpt-4

# 设置温度
codex --temperature 0.2

# 限制 token 使用
codex --max-tokens 2000

# 指定文件上下文
codex --context-file src/auth.js

# 忽略文件
codex --ignore "**/*.test.js"
```

---

## 使用模式

### 1. 交互式模式

```bash
# 启动交互会话
codex

# 在会话中
> Help me understand this codebase
> Now add error handling
> Run the tests
```

### 2. 批处理模式

```bash
# 从文件读取提示
codex -f prompts.txt

# 管道输入
echo "Fix linting errors" | codex
```

### 3. 编辑器集成

```bash
# VS Code 扩展
# 安装 Codex 扩展后使用 Cmd/Ctrl+I

# JetBrains 插件
# 在 IDE 设置中启用 Codex
```

---

## 配置选项

### 配置文件位置

```bash
# macOS/Linux
~/.codex/config.yaml

# Windows
%USERPROFILE%\.codex\config.yaml
```

### 配置示例

```yaml
model: gpt-4
temperature: 0.2
max_tokens: 4096
auto_accept: false
max_retries: 3

# 上下文配置
context:
  include:
    - "**/*.js"
    - "**/*.ts"
  exclude:
    - "**/*.test.js"
    - "node_modules/**"
    - "**/*.min.js"

# 工具配置
tools:
  bash: true
  read: true
  write: true
  edit: false  # 需要确认

# Hook 配置
hooks:
  pre:
    - "echo 'Starting Codex session'"
  post:
    - "echo 'Session complete'"
```

### 环境变量

```bash
# API 配置
export OPENAI_API_KEY="sk-..."
export OPENAI_ORG_ID="org-..."

# 模型配置
export CODEX_MODEL="gpt-4"
export CODEX_TEMPERATURE="0.2"

# 行为配置
export CODEX_AUTO_ACCEPT="false"
export CODEX_MAX_RETRIES="3"
```

---

## 最佳实践

### 1. 明确提示

```
✓ "在 src/auth.js 中添加 JWT 验证中间件"
✓ "重构 calculateTotal() 函数，使用 reduce"
✓ "为 UserService 添加单元测试"

✗ "改进代码"
✗ "修复问题"
```

### 2. 提供上下文

```bash
# 指定相关文件
codex --context-file src/user.js --context-file tests/user.test.js

# 或一次性提供
codex "Review the auth flow in src/auth.js and src/middleware.js"
```

### 3. 增量修改

```bash
# 小步快跑
codex "Add input validation to login function"
codex "Add error handling for database errors"
codex "Write unit tests for login function"
```

### 4. 安全设置

```yaml
# 限制危险操作
tools:
  bash: true
  write: false  # 禁止直接写入
  edit: true    # 需要确认
```

---

## 与 Claude Code 对比

| 特性 | Claude Code | Codex CLI |
|------|-------------|-----------|
| 模型 | Claude (Sonnet/Opus) | GPT-4/Codex |
| 上下文窗口 | 1M+ tokens | 128K-200K tokens |
| 技能系统 | 丰富的技能市场 | 较少扩展 |
| 智能体 | 多智能体协作 | 单智能体 |
| Hooks | 完整的 Hooks 系统 | 基础 Hooks |
| MCP 支持 | 支持 | 不支持 |
| 记忆系统 | 长期记忆 | 无 |
| 成本 | 相对较高 | 相对经济 |

### 选择建议

**使用 Claude Code 当：**
- 需要复杂的多智能体协作
- 项目需要长期记忆和配置
- 需要丰富的技能和扩展
- 处理大型代码库 (1M+ tokens)

**使用 Codex CLI 当：**
- 需要快速编辑和修改
- 与 OpenAI 生态集成
- 成本敏感
- 简单的批处理任务

---

## 高级用法

### 自定义 Prompt 模板

```bash
# 创建模板文件
codex -t templates/my-template.md

# 模板内容
## Context
{{file_content}}

## Task
{{user_prompt}}

## Output Format
1. Analysis
2. Changes
3. Tests
```

### 批量处理

```bash
# 处理多个文件
for file in src/*.js; do
  codex --file $file "Add JSDoc comments"
done

# 使用脚本
codex -f batch-prompts.txt
```

### Git 集成

```bash
# 查看变更
codex "Review git diff"

# 提交前检查
codex "Check this commit for issues"

# 生成提交信息
codex --suggestions "Generate commit message"
```

---

## 常见问题

### Q: 如何限制 Codex 只能读取不能写入？
```yaml
tools:
  write: false
  edit: false
```

### Q: 如何添加自定义系统提示？
```bash
codex --system-prompt "You are a senior React developer..."
```

### Q: 如何处理大型代码库？
```bash
# 排除无关文件
codex --ignore "node_modules/**" "**/*.test.js"

# 使用上下文窗口
codex --context-file src/main.js
```

---

## 参考资料

- [OpenAI Codex GitHub](https://github.com/openai/codex)
- [OpenAI API 文档](https://platform.openai.com/docs)
- [Codex CLI 文档](https://platform.openai.com/docs/codex)

---

## 相关资源

- [OpenAI Codex 官方文档](https://platform.openai.com/docs/codex/overview)
- [OpenAI GitHub 仓库](https://github.com/openai/codex)
- [OpenAI API 参考](https://platform.openai.com/docs/api-reference)
- [模型选择指南](https://platform.openai.com/docs/models)
