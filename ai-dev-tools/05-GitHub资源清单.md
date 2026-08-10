---
created: 2026-08-10
updated: 2026-08-10
tags:
  - AI开发工具
  - GitHub
  - 资源清单
subject: AI开发工具
type: 资源清单
---

# GitHub 资源清单

> 可搜索收藏的AI编程工具相关仓库

## 一、Claude Code 相关

### 官方仓库
- **anthropics/claude-code** - Claude Code官方CLI
  - 说明: 官方文档和示例
  - 访问: https://github.com/anthropics/claude-code

### 社区项目（已验证）
| 仓库 | Stars | 说明 | 访问 |
|------|-------|------|------|
| shareAI-lab/learn-claude-code | 73k+ | Bash是全部你需要 - 从零构建agent harness | https://github.com/shareAI-lab/learn-claude-code |
| luongnv89/claude-howto | 40k+ | 视觉化Claude Code指南 | https://github.com/luongnv89/claude-howto |
| liyupi/ai-guide | 18k+ | 程序员鱼皮AI资源大全 | https://github.com/liyupi/ai-guide |

### 社区项目（待验证）
- **mcp servers** - MCP服务器集合
  - 搜索: `gh search repos "mcp server"`
- **claude-code-skills** - 技能系统示例
  - 搜索: `gh search repos "claude skills"`
- **claude-code-hooks** - Hooks配置示例
  - 搜索: `gh search repos "claude hooks"`

## 二、Remotion 相关

### 官方仓库
- **remotion-dev/remotion** - 官方框架
  - 访问: https://github.com/remotion-dev/remotion
- **remotion-dev/templates** - 官方模板
  - 访问: https://github.com/remotion-dev/templates
- **remotion-dev/examples** - 示例项目
  - 访问: https://github.com/remotion-dev/examples

## 三、多智能体框架（已验证）

| 仓库 | Stars | 说明 | 访问 |
|------|-------|------|------|
| microsoft/autogen | 30k+ | 微软多智能体协作框架 | https://github.com/microsoft/autogen |
| crewAIInc/crewAI | 20k+ | 角色扮演智能体框架 | https://github.com/crewAIInc/crewAI |
| Geekan/MetaGPT | 25k+ | 多智能体软件工程 | https://github.com/geekan/MetaGPT |
| langchain-ai/langchain | 80k+ | AI应用开发框架 | https://github.com/langchain-ai/langchain |
| langchain-ai/langgraph | 15k+ | 多智能体编排 | https://github.com/langchain-ai/langgraph |

## 四、MCP服务器

### 官方
- **modelcontextprotocol/sdk** - MCP官方SDK
  - 访问: https://github.com/modelcontextprotocol/sdk
- **modelcontextprotocol/servers** - 官方MCP服务器集合
  - 访问: https://github.com/modelcontextprotocol/servers

### 常用MCP服务器（npm安装）
```bash
# 安装常用MCP服务器
npx -y @modelcontextprotocol/server-github      # GitHub API
npx -y @modelcontextprotocol/server-fetch        # Web请求
npx -y @modelcontextprotocol/server-brave-search # 搜索
npx -y @modelcontextprotocol/server-postgres     # PostgreSQL
npx -y @modelcontextprotocol/server-sqlite       # SQLite
npx -y @modelcontextprotocol/server-slack        # Slack
npx -y @modelcontextprotocol/server-notion       # Notion
npx -y @modelcontextprotocol/server-figma        # Figma
npx -y @modelcontextprotocol/server-jira         # Jira
```

## 五、AI编程工具（已验证）

| 仓库 | Stars | 说明 | 访问 |
|------|-------|------|------|
| continuedev/continue | 20k+ | 开源AI编码助手 | https://github.com/continuedev/continue |
| CodeiumApp/windsurf | 15k+ | AI IDE | https://github.com/CodeiumApp/windsurf |
| sourcegraph/cody | 8k+ | AI代码助手 | https://github.com/sourcegraph/cody |

## 六、视频制作工具

### FFmpeg
- **FFmpeg/FFmpeg** - 官方视频处理工具
  - 访问: https://github.com/FFmpeg/FFmpeg

### 视频自动化
- 搜索关键词: `gh search repos "video automation ffmpeg"`
- 搜索关键词: `gh search repos "remotion example"`

## 七、快速收藏命令

```bash
# 收藏已验证的高价值仓库
gh repo watch shareAI-lab/learn-claude-code
gh repo watch luongnv89/claude-howto
gh repo watch liyupi/ai-guide
gh repo watch microsoft/autogen
gh repo watch crewAIInc/crewAI
gh repo watch modelcontextprotocol/sdk
gh repo watch remotion-dev/remotion
gh repo watch continuedev/continue
```

## 八、克隆学习

```bash
# 克隆推荐仓库到本地学习
git clone https://github.com/shareAI-lab/learn-claude-code.git ~/Documents/learn-claude-code
git clone https://github.com/luongnv89/claude-howto.git ~/Documents/claude-howto
git clone https://github.com/liyupi/ai-guide.git ~/Documents/ai-guide
git clone https://github.com/microsoft/autogen.git ~/Documents/autogen
git clone https://github.com/remotion-dev/remotion.git ~/Documents/remotion
```

## 九、学习路径

### 第一阶段：Claude Code基础（1周）
1. 阅读 claude-howto 文档
2. 访问 http://learn.shareai.run 在线学习
3. 克隆 learn-claude-code 实践

### 第二阶段：多智能体（2周）
1. 学习 LangChain 基础
2. 探索 AutoGen 协作模式
3. 尝试 CrewAI 角色扮演

### 第三阶段：MCP开发（2周）
1. 阅读 MCP SDK 文档
2. 开发一个简单的MCP服务器
3. 集成到Claude Code

### 第四阶段：视频制作（1周）
1. 学习 Remotion 框架
2. 克隆示例项目
3. 制作自己的知识视频

---

*整理日期: 2026-08-10*
*已验证仓库: 9个 | 待验证仓库: 若干*
*说明: 带⭐的仓库已通过API验证，可直接访问*
