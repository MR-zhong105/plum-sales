# Claude Code Hooks 系统完全指南

> 最后更新: 2026-08-09

## 目录
- [Hooks 简介](#hooks-简介)
- [Hook 类型](#hook-类型)
- [配置方法](#配置方法)
- [常用场景](#常用场景)
- [高级用法](#高级用法)
- [调试技巧](#调试技巧)

---

## Hooks 简介

Hooks 是 Claude Code 的自动化触发机制，允许在特定事件时自动执行 shell 命令。

**核心价值：**
- 自动代码格式化
- 运行 linter
- 执行安全检查
- 记录审计日志
- 自动提交

---

## Hook 类型

### 1. PreToolUse Hook

在工具执行前触发。

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          "echo \"[Pre-Bash] Running: $TOOL_INPUT\"",
          "test -f .gitignore && echo 'gitignore exists'"
        ]
      },
      {
        "matcher": "Edit",
        "hooks": [
          "echo \"[Pre-Edit] File: $TOOL_FILE_PATH\""
        ]
      }
    ]
  }
}
```

**可用变量：**
- `$TOOL_NAME` - 工具名称
- `$TOOL_INPUT` - 输入参数
- `$TOOL_FILE_PATH` - 文件路径 (Edit/Read 工具)

### 2. PostToolUse Hook

在工具执行后触发。

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          "echo \"[Post-Bash] Exit code: $EXIT_CODE\"",
          "if [ $EXIT_CODE -ne 0 ]; then echo 'Command failed'; fi"
        ]
      },
      {
        "matcher": "Write",
        "hooks": [
          "echo \"[Post-Write] File saved: $TOOL_FILE_PATH\""
        ]
      }
    ]
  }
}
```

**可用变量：**
- `$EXIT_CODE` - 退出代码
- `$TOOL_OUTPUT` - 输出内容
- `$TOOL_FILE_PATH` - 文件路径

### 3. Stop Hook

在会话结束时触发。

```json
{
  "hooks": {
    "Stop": [
      "echo \"[Session End] $(date)\"",
      "git status --short > /tmp/claude-code-session.log",
      "open /tmp/claude-code-session.log"
    ]
  }
}
```

---

## 配置方法

### 全局配置 (~/.claude/settings.json)

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "*",
        "hooks": [
          "echo \"[Global Pre] Tool: $TOOL_NAME\""
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "*",
        "hooks": [
          "echo \"[Global Post] Completed\""
        ]
      }
    ]
  }
}
```

### 项目配置 (.claude/settings.json)

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash(npm *)",
        "hooks": [
          "npm run lint"
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit",
        "hooks": [
          "prettier --write $TOOL_FILE_PATH"
        ]
      }
    ]
  }
}
```

### 匹配器语法

```json
// 精确匹配
"matcher": "Bash"

// 前缀匹配
"matcher": "Bash(npm *)"

// 通配符匹配
"matcher": "Bash(*)"

// 多个匹配器
"matcher": ["Bash(npm *)", "Bash(yarn *)"]

// 匹配所有
"matcher": "*"
```

---

## 常用场景

### 1. 自动格式化

```json
{
  "PostToolUse": [
    {
      "matcher": "Write",
      "hooks": [
        "prettier --write $TOOL_FILE_PATH",
        "eslint --fix $TOOL_FILE_PATH"
      ]
    },
    {
      "matcher": "Edit",
      "hooks": [
        "prettier --write $TOOL_FILE_PATH",
        "eslint --fix $TOOL_FILE_PATH"
      ]
    }
  ]
}
```

### 2. 运行测试

```json
{
  "PostToolUse": [
    {
      "matcher": "Bash(npm test *)",
      "hooks": [
        "npm test -- --watchAll=false"
      ]
    }
  ]
}
```

### 3. Git 自动提交

```json
{
  "Stop": [
    "if git status --porcelain; then git add -A && git commit -m 'Auto-commit from Claude Code'; fi"
  ]
}
```

### 4. 安全检查

```json
{
  "PreToolUse": [
    {
      "matcher": "Bash(rm *)",
      "hooks": [
        "echo 'WARNING: Delete command detected'",
        "test -f .allow-delete && echo 'Proceeding...'"
      ]
    }
  ]
}
```

### 5. 构建验证

```json
{
  "PostToolUse": [
    {
      "matcher": "Write",
      "hooks": [
        "npm run build",
        "npm run test"
      ]
    }
  ]
}
```

### 6. 审计日志

```json
{
  "PostToolUse": [
    {
      "matcher": "*",
      "hooks": [
        "echo \"$(date): $TOOL_NAME - $TOOL_INPUT\" >> /tmp/claude-audit.log"
      ]
    }
  ]
}
```

---

## 高级用法

### 条件执行

```json
{
  "PostToolUse": [
    {
      "matcher": "Bash",
      "hooks": [
        "if [ \"$TOOL_INPUT\" = \"npm test\" ]; then npm test; fi",
        "case \"$TOOL_INPUT\" in *build*) npm run build;; esac"
      ]
    }
  ]
}
```

### 并行执行

```json
{
  "PostToolUse": [
    {
      "matcher": "Write",
      "hooks": [
        "prettier --write $TOOL_FILE_PATH &",
        "eslint --fix $TOOL_FILE_PATH &",
        "wait"
      ]
    }
  ]
}
```

### 错误处理

```json
{
  "PostToolUse": [
    {
      "matcher": "Bash",
      "hooks": [
        "if [ $EXIT_CODE -ne 0 ]; then",
        "  echo 'Command failed, retrying...'",
        "  sleep 2",
        "  $TOOL_INPUT",
        "fi"
      ]
    }
  ]
}
```

### 文件监控

```json
{
  "PostToolUse": [
    {
      "matcher": "Write",
      "hooks": [
        "inotifywait -q -e modify $TOOL_FILE_PATH &>/dev/null &",
        "npm run dev"
      ]
    }
  ]
}
```

---

## 调试技巧

### 1. 启用详细日志

```bash
# 设置调试模式
export CLAUDE_CODE_DEBUG=true

# 查看详细日志
claude --debug
```

### 2. 测试 Hook

```bash
# 手动执行 hook 脚本
bash -c 'echo "Test hook: $TOOL_NAME"'

# 验证退出代码
echo $?
```

### 3. 常见问题

| 问题 | 解决方案 |
|------|---------|
| Hook 不执行 | 检查匹配器语法 |
| Hook 执行慢 | 优化命令，避免阻塞 |
| 变量未定义 | 检查变量名是否正确 |
| 权限错误 | 确保脚本有执行权限 |

### 4. 调试配置

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "*",
        "hooks": [
          "echo \"[DEBUG] Tool: $TOOL_NAME\" >> /tmp/hooks-debug.log",
          "echo \"[DEBUG] Input: $TOOL_INPUT\" >> /tmp/hooks-debug.log"
        ]
      }
    ]
  }
}
```

---

## 最佳实践

### 1. 避免阻塞

```json
// 错误：阻塞式操作
"hooks": ["npm test"]

// 正确：后台运行
"hooks": ["npm test &"]
```

### 2. 快速失败

```json
"hooks": [
  "if ! npm run lint; then exit 1; fi"
]
```

### 3. 日志轮转

```json
"hooks": [
  "echo \"$(date): $TOOL_NAME\" >> /tmp/hooks.log && find /tmp -name '*.log' -mtime +7 -delete"
]
```

### 4. 错误隔离

```json
"hooks": [
  "set -e",  // 遇到错误立即退出
  "trap 'echo Hook failed' ERR"
]
```

---

## 参考资料

- [Claude Code Hooks 文档](https://docs.anthropic.com/en/docs/claude-code/hooks)
- [Hooks 配置示例](https://github.com/anthropics/claude-code/tree/main/hooks)

---

## 相关资源

- [Anthropic Hooks 官方文档](https://docs.anthropic.com/en/docs/claude-code/hooks)
- [Hook 最佳实践](https://docs.anthropic.com/en/docs/claude-code/hooks-best-practices)
- [社区 Hook 示例](https://github.com/anthropics/claude-code/discussions/hooks)
