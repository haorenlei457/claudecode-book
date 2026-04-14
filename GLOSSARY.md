# 📖 Claude Code 词汇表

## 📚 快速导航

- [核心概念](#核心概念)
- [插件系统](#插件系统)
- [命令系统](#命令系统)
- [代理与技能](#代理与技能)
- [配置与环境](#配置与环境)
- [Git与版本控制](#git与版本控制)
- [安全与权限](#安全与权限)

---

## 核心概念

### Agent (代理)
- **定义**：具备特定任务能力的 AI 助手
- **特点**：专业化、可委派、有系统提示词
- **示例**：code-reviewer, test-engineer, documentation-writer
- **相关**：[04-代理系统](./04-agent-system.md)

### Context (上下文)
- **定义**：Claude Code 对项目的整体理解
- **组成**：文件结构、代码依赖、项目配置、历史记录
- **加载优先级**：当前目录 → 父目录 → 全局配置
- **相关**：[09-文件操作与上下文管理](./09-file-context.md)

### MCP (Model Context Protocol)
- **定义**：模型上下文协议，用于连接外部工具和服务
- **作用**：让 Claude Code 访问 GitHub、数据库、文件系统等
- **示例**：github-mcp, filesystem-mcp, database-mcp
- **相关**：[07-MCP协议集成](./07-mcp-protocol.md)

### Plugin (插件)
- **定义**：扩展 Claude Code 功能的模块化组件
- **组成**：commands + agents + skills + hooks
- **类型**：官方插件、社区插件、自定义插件
- **相关**：[02-插件系统](./02-plugin-system.md)

### Skill (技能)
- **定义**：封装的可复用能力包
- **触发**：自动触发或手动调用
- **类型**：code-review, test-generation, documentation
- **相关**：[05-技能系统](./05-skill-system.md)

---

## 插件系统

### Agent (代理)
参见上文 [Agent (代理)](#agent-代理)

### Hook (钩子)
- **定义**：事件驱动的自动化脚本
- **事件类型**：SessionStart, PreToolUse, PostToolUse, Stop
- **用途**：预检查、后处理、自动化、监控
- **相关**：[06-钩子系统](./06-hook-system.md)

### Manifest (插件清单)
- **定义**：插件的元数据文件（`plugin.json`）
- **内容**：name, version, description, exports, permissions
- **位置**：`.claude-plugin/plugin.json`
- **验证**：必须包含必需字段且格式正确

### Plugin Marketplace (插件市场)
- **定义**：集中管理和分发插件的平台
- **作用**：发现、安装、管理插件
- **官方市场**：Anthropic 官方插件目录

---

## 命令系统

### Alias (别名)
- **定义**：命令的简短名称
- **示例**：`/ci` 是 `/commit` 的别名
- **配置**：在命令的 frontmatter 中定义

### Argument (参数)
- **定义**：传递给命令的值
- **类型**：
  - **位置参数**：按顺序传递 (`/cmd arg1 arg2`)
  - **命名参数**：使用选项名 (`/cmd --opt value`)
- **访问**：`{{args.0}}` 或 `{{options.opt}}`

### Command (命令)
- **定义**：以 `/` 开头的快捷指令
- **结构**：frontmatter + Markdown 内容
- **触发**：用户输入或自动触发
- **相关**：[03-命令系统](./03-command-system.md)

### Frontmatter
- **定义**：Markdown 文件顶部的元数据块
- **格式**：YAML 格式，用 `---` 包围
- **用途**：定义名称、描述、选项等

### Matcher (匹配器)
- **定义**：用于匹配命令或触发钩子的模式
- **类型**：精确匹配、模糊匹配、正则表达式
- **示例**：`"Write"` 匹配写入文件操作

---

## 代理与技能

### Agent Orchestration (代理编排)
- **定义**：协调多个代理协作完成复杂任务
- **模式**：串行、并行、主从
- **示例**：开发新功能时的代理链
- **相关**：[04-代理系统](./04-agent-system.md)

### Capability (能力)
- **定义**：代理或技能具备的功能
- **列表**：在元数据中声明
- **用途**：代理选择和任务委派

### Skill (技能)
参见上文 [Skill (技能)](#skill-技能)

### Subagent (子代理)
- **定义**：在主代理控制下执行特定任务的代理
- **特点**：继承主代理的上下文和权限
- **用途**：任务分解和并行执行

### System Prompt (系统提示词)
- **定义**：代理的核心行为指令
- **内容**：角色、职责、工作流程、输出格式
- **重要性**：决定了代理的行为和质量

---

## 配置与环境

### Environment Variable (环境变量)
- **定义**：操作系统中存储配置信息的方式
- **用途**：存储 API 密钥、路径、配置选项
- **示例**：`ANTHROPIC_API_KEY`, `PATH`

### Global Config (全局配置)
- **定义**：影响所有项目的配置
- **位置**：`~/.claude/settings.json`
- **优先级**：低于项目配置

### Project Config (项目配置)
- **定义**：仅影响当前项目的配置
- **位置**：`./.claude/settings.json`
- **优先级**：最高

### Settings (设置)
- **定义**：控制 Claude Code 行为的配置项
- **类型**：model, maxTokens, temperature, plugins, hooks
- **相关**：[08-配置系统](./08-configuration.md)

---

## Git与版本控制

### Branch (分支)
- **定义**：Git 中的独立开发线
- **类型**：main, develop, feature/*, hotfix/*
- **用途**：并行开发和版本隔离

### Conventional Commits
- **定义**：提交信息规范
- **格式**：`<type>(<scope>): <subject>`
- **类型**：feat, fix, docs, style, refactor, test, chore
- **相关**：[10-Git集成](./10-git-integration.md)

### Diff (差异)
- **定义**：文件或提交之间的变更
- **类型**：工作区差异、暂存区差异、提交差异
- **用途**：查看变更内容

### Pull Request (PR)
- **定义**：合并代码变更的请求
- **流程**：创建 → 审查 → 修改 → 合并
- **相关**：[10-Git集成](./10-git-integration.md)

### Stage (暂存)
- **定义**：将文件变更添加到暂存区
- **命令**：`git add`
- **用途**：准备提交的变更

---

## 安全与权限

### Permission (权限)
- **定义**：允许或禁止特定操作的规则
- **类型**：file:read, file:write, bash:run, git:write
- **配置**：在 `settings.json` 中定义

### RBAC (Role-Based Access Control)
- **定义**：基于角色的访问控制
- **概念**：为代理分配角色，角色决定权限
- **用途**：精细化的权限管理

### Sandbox (沙箱)
- **定义**：隔离插件和代理执行环境的机制
- **作用**：防止恶意代码影响系统
- **特性**：受限的 API、独立的存储、事件总线

### Security Hook (安全钩子)
- **定义**：用于安全检查的钩子
- **类型**：敏感信息检测、危险命令拦截
- **示例**：检查 API 密钥泄露、阻止 `rm -rf /`
- **相关**：[12-安全机制](./12-security.md)

---

## 其他术语

### CLI (Command Line Interface)
- **定义**：命令行界面
- **特点**：基于文本的交互方式
- **Claude Code**：运行在 CLI 的 AI 编程助手

### LLM (Large Language Model)
- **定义**：大语言模型
- **示例**：Claude, GPT, Llama
- **Claude Code**：使用 Claude 系列模型

### Token
- **定义**：模型处理文本的基本单位
- **影响**：决定输入和输出的长度限制
- **配置**：`maxTokens` 参数控制

### Workspace (工作区)
- **定义**：当前项目的根目录
- **上下文**：所有操作相对于工作区进行
- **切换**：`cd` 命令改变工作区

---

## 📖 学习建议

1. **从概念开始**：先理解核心概念
2. **结合实践**：通过实际使用加深理解
3. **查阅文档**：每个术语都有对应的详细文档
4. **随时查阅**：遇到不熟悉的术语立即查找

---

**提示**：使用浏览器搜索（Ctrl+F / Cmd+F）快速查找术语！🔍
