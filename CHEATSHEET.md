# ⚡ Claude Code 速查手册

## 🎯 常用命令快速参考

### 基本操作

```bash
# 启动
claude

# 查看版本
claude --version

# 测试连接
claude --ping

# 查看帮助
claude --help
```

### 插件管理

```bash
# 列出插件
claude> /plugins list

# 安装插件
claude> /plugin install code-review

# 禁用插件
claude> /plugin disable plugin-name

# 启用插件
claude> /plugin enable plugin-name

# 更新插件
claude> /plugin update plugin-name
```

### Git操作

```bash
# 智能提交
claude> /commit

# 查看状态
claude> git status

# 创建分支
claude> git checkout -b feature/name

# 创建PR
claude> /pr create

# 审查PR
claude> /pr review #123

# 合并PR
claude> /pr merge #123
```

### 配置管理

```bash
# 显示配置
claude> /config show

# 重置配置
claude> /config reset

# 验证配置
claude> /config validate
```

---

## 📦 插件速查

### 官方插件

| 插件名 | 功能 | 命令 |
|--------|------|------|
| **code-review** | 代码审查 | `/code-review` |
| **commit-commands** | Git命令 | `/commit`, `/pr` |
| **feature-dev** | 功能开发 | `/feature-dev` |
| **plugin-dev** | 插件开发 | `/plugin-dev` |
| **hookify** | Hook管理 | `/hookify` |

### 常用插件配置

```json
{
  "plugins": [
    "code-review",
    "commit-commands",
    "feature-dev"
  ]
}
```

---

## 🎨 命令速查

### 斜杠命令

```bash
/help                    # 显示帮助
/commands list           # 列出所有命令
/plugins list            # 列出插件
/config show             # 显示配置
/version                 # 显示版本
/history                 # 查看历史
/clear                   # 清屏
/exit                    # 退出
```

### 文件操作

```bash
# 读取文件
cat file.txt
read file.md

# 写入文件
echo "content" > file.txt
write file.txt "content"

# 编辑文件
edit file.py "添加日志"
```

### Git操作

```bash
# 提交
commit "message"
commit -m "message"

# 推送
push
push origin main

# 拉取
pull
pull origin main

# 分支
branch feature/name
checkout feature/name
checkout -b feature/name
```

---

## ⚙️ 配置文件模板

### settings.json

```json
{
  "model": "claude-sonnet-4",
  "maxTokens": 8192,
  "temperature": 0.7,
  "plugins": ["code-review"],
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write",
        "hooks": [".claude/hooks/pre-write.sh"]
      }
    ]
  },
  "permissions": {
    "file:read": true,
    "file:write": ["./**"],
    "bash:run": true
  }
}
```

### CLAUDE.md

```markdown
# 项目名称

## 技术栈
- React + TypeScript
- Node.js + Express

## 编码规范
- ESLint + Prettier
- 类型注解必须

## Git工作流
- feature/* 分支
- Conventional Commits
- PR需要审查
```

---

## 🤝 代理速查

### 常用代理

| 代理ID | 名称 | 用途 |
|--------|------|------|
| `code-reviewer` | 代码审查员 | 审查代码质量 |
| `test-engineer` | 测试工程师 | 编写测试 |
| `documentation-writer` | 文档编写者 | 生成文档 |
| `security-auditor` | 安全审计员 | 安全检查 |

### 使用代理

```bash
# 自动委派
claude> 审查这个文件

# 手动指定
claude> @code-reviewer 审查 main.py

# 多代理协作
claude> 完成用户认证功能
```

---

## 🔌 Hook 速查

### Hook事件

| 事件 | 触发时机 | 用途 |
|------|----------|------|
| **SessionStart** | 会话开始 | 初始化、加载配置 |
| **PreToolUse** | 工具使用前 | 验证权限、记录日志 |
| **PostToolUse** | 工具使用后 | 清理、更新状态 |
| **PreResponse** | 响应前 | 审查输出、格式化 |
| **Stop** | 停止时 | 保存状态、通知 |

### Hook配置

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "*",
        "hooks": ["init.sh"]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Write",
        "hooks": ["pre-write.sh"]
      }
    ]
  }
}
```

---

## 🔐 安全速查

### 权限类型

| 权限 | 说明 | 示例 |
|------|------|------|
| `file:read` | 读取文件 | `["./src/**"]` |
| `file:write` | 写入文件 | `["./src/**", "./tests/**"]` |
| `file:delete` | 删除文件 | `["./temp/**"]` |
| `bash:run` | 执行命令 | `true` |
| `git:read` | Git读取 | `true` |
| `git:write` | Git写入 | `true` |

### 安全配置

```json
{
  "permissions": {
    "file:write": ["./src/**"],
    "bash:run": true,
    "bash:allowedCommands": ["npm", "git", "python"]
  }
}
```

---

## 🌐 MCP速查

### 常用MCP服务器

| 服务器 | 功能 | 安装 |
|--------|------|------|
| **github** | GitHub集成 | `npx @modelcontextprotocol/server-github` |
| **filesystem** | 文件系统 | `npx @modelcontextprotocol/server-filesystem` |
| **database** | 数据库 | `npx @modelcontextprotocol/server-postgres` |
| **brave-search** | 搜索 | `npx @modelcontextprotocol/server-brave-search` |

### MCP配置

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

---

## 📁 文件结构速查

### 配置目录结构

```
~/.claude/                      # 全局配置
├── settings.json              # 核心配置
├── CLAUDE.md                  # 个人偏好
├── commands/                  # 全局命令
├── agents/                    # 全局代理
├── skills/                    # 全局技能
└── hooks/                     # 全局钩子

./.claude/                     # 项目配置
├── settings.json              # 项目配置
├── CLAUDE.md                  # 项目上下文
└── plugins/                   # 项目插件
```

### 文件路径

```bash
~/.claude/settings.json        # 全局配置
./.claude/settings.json        # 项目配置
./CLAUDE.md                    # 项目根上下文
./src/CLAUDE.md                # 源代码上下文
```

---

## 🎯 快捷键速查

### 终端快捷键

| 快捷键 | 功能 |
|--------|------|
| `Ctrl+C` | 中断当前操作 |
| `Ctrl+D` | 退出Claude |
| `Ctrl+L` | 清屏 |
| `↑/↓` | 浏览历史命令 |
| `Tab` | 自动补全 |

### 命令快捷方式

```bash
/c        # /commit
/p        # /pr
/r        # /review
/t        # /test
/d        # /deploy
```

---

## 🐛 故障排查速查

### 常见问题速查

| 问题 | 解决方案 |
|------|----------|
| 命令未找到 | `export PATH="$HOME/.local/bin:$PATH"` |
| API密钥无效 | 检查 `ANTHROPIC_API_KEY` 环境变量 |
| 权限不足 | 检查 `settings.json` 中的 permissions |
| 插件不工作 | `claude> /plugin validate plugin-name` |
| 网络连接失败 | 检查代理设置和防火墙 |
| 配置冲突 | `claude> /config show` 查看最终配置 |

### 调试命令

```bash
# 启用调试模式
claude --debug

# 查看日志
claude> /logs

# 验证配置
claude> /config validate

# 测试连接
claude --ping
```

---

## 📚 更多资源

### 官方资源
- [Claude Code 官方文档](https://code.claude.com/docs/en/overview)
- [GitHub 仓库](https://github.com/anthropics/claude-code)
- [Discord 社区](https://anthropic.com/discord)

### 本地资源
- [FAQ](./FAQ.md) - 常见问题解答
- [词汇表](./GLOSSARY.md) - 术语解释
- [环境搭建](./SETUP_GUIDE.md) - 安装配置
- [实战项目](./PRACTICE_PROJECTS.md) - 项目示例

---

**提示**：打印此文档作为桌面速查参考！🖨️
