# 🚀 Claude Code 环境搭建指南

## 📋 快速开始

本指南将帮助你在各种操作系统上安装和配置 Claude Code。

---

## 🖥️ 系统要求

### 最低配置
- **操作系统**: macOS 10.15+, Ubuntu 18.04+, Windows 10+
- **内存**: 4GB RAM
- **磁盘**: 1GB 可用空间
- **网络**: 稳定的互联网连接

### 推荐配置
- **内存**: 8GB+ RAM
- **Node.js**: 18.x LTS 或更高

---

## 📦 安装方法

### macOS / Linux (推荐)

#### 方法1：官方安装脚本

```bash
# 使用 curl
curl -fsSL https://claude.ai/install.sh | bash

# 或使用 wget
wget -qO- https://claude.ai/install.sh | bash
```

#### 方法2：Homebrew

```bash
# 添加 tap
brew tap anthropic/tap

# 安装
brew install claude-code

# 更新
brew upgrade claude-code
```

### Windows

#### 方法1：PowerShell 脚本

```powershell
# 以管理员身份运行 PowerShell
irm https://claude.ai/install.ps1 | iex
```

#### 方法2：WinGet

```powershell
winget install Anthropic.ClaudeCode
```

#### 方法3：手动安装

1. 下载 Windows 安装包：[下载链接]
2. 运行安装程序
3. 按照向导完成安装

### Linux (手动安装)

```bash
# 下载最新版本
wget https://github.com/anthropics/claude-code/releases/latest/download/claude-code-linux.tar.gz

# 解压
tar -xzf claude-code-linux.tar.gz

# 移动到 PATH
sudo mv claude /usr/local/bin/

# 验证安装
claude --version
```

---

## ⚙️ 初始配置

### 1. 首次运行

```bash
$ claude

# 首次运行会提示：
🎉 欢迎使用 Claude Code！

请完成以下设置：
1. API 密钥配置
2. 偏好设置
3. 插件选择
```

### 2. API 密钥配置

#### 获取 API 密钥

1. 访问 [Anthropic Console](https://console.anthropic.com)
2. 创建 API 密钥
3. 复制密钥

#### 配置密钥

```bash
# 方法1：环境变量
export ANTHROPIC_API_KEY="your-api-key-here"

# 添加到 ~/.bashrc 或 ~/.zshrc
echo 'export ANTHROPIC_API_KEY="your-api-key-here"' >> ~/.bashrc
source ~/.bashrc

# 方法2：配置文件
mkdir -p ~/.config/claude
echo '{"apiKey": "your-api-key-here"}' > ~/.config/claude/config.json
```

### 3. 验证安装

```bash
# 检查版本
claude --version

# 测试连接
claude --ping

# 查看帮助
claude --help
```

---

## 🔧 高级配置

### 配置文件结构

```
~/.claude/
├── settings.json          # 核心配置
├── CLAUDE.md              # 个人偏好
├── commands/              # 自定义命令
├── agents/                # 自定义代理
├── skills/                # 自定义技能
└── hooks/                 # 自定义钩子
```

### 推荐配置

```json
// ~/.claude/settings.json
{
  "model": "claude-sonnet-4",
  "maxTokens": 8192,
  "temperature": 0.7,
  "output": {
    "style": "concise",
    "includeTimestamps": true
  },
  "plugins": [
    "code-review",
    "commit-commands"
  ],
  "permissions": {
    "file:read": true,
    "file:write": ["./**"],
    "bash:run": true
  }
}
```

### CLAUDE.md 模板

```markdown
# 个人偏好配置

## 编码风格
- 偏好简洁的代码
- 注重类型安全
- 使用函数式编程

## 沟通方式
- 直接、简洁的回答
- 提供代码示例
- 解释"为什么"

## 常用技术栈
- TypeScript / JavaScript
- Python
- React / Node.js
```

---

## 🐛 常见问题排查

### 问题1：命令未找到

```bash
# 症状：zsh: command not found: claude

# 解决方案1：检查 PATH
echo $PATH | grep claude

# 解决方案2：手动添加
export PATH="$HOME/.local/bin:$PATH"

# 解决方案3：重新安装
# 按照官方安装步骤重新安装
```

### 问题2：API 密钥无效

```bash
# 症状：Error: Invalid API key

# 检查环境变量
echo $ANTHROPIC_API_KEY

# 验证密钥格式
# 应该以 "sk-ant-" 开头

# 重新配置
export ANTHROPIC_API_KEY="your-correct-key"
```

### 问题3：权限不足

```bash
# 症状：Permission denied

# 检查文件权限
ls -la $(which claude)

# 修复权限
chmod +x /usr/local/bin/claude

# 或使用 sudo
sudo claude
```

### 问题4：网络连接失败

```bash
# 症状：Network error / Connection timeout

# 检查网络
ping anthropic.com

# 检查代理设置
echo $HTTP_PROXY
echo $HTTPS_PROXY

# 临时禁用代理
unset HTTP_PROXY
unset HTTPS_PROXY
```

### 问题5：更新失败

```bash
# 症状：Update failed

# 手动更新
# macOS/Linux
curl -fsSL https://claude.ai/install.sh | bash

# Windows
irm https://claude.ai/install.ps1 | iex

# 或使用包管理器
brew upgrade claude-code
```

---

## 🔌 IDE 集成

### VS Code

1. 安装 Claude Code 扩展
2. 配置 API 密钥
3. 使用快捷键 `Ctrl+Shift+C` 打开

### JetBrains (IntelliJ, PyCharm)

1. 安装 Claude 插件
2. 在设置中配置
3. 右键菜单中使用

### Vim / Neovim

```vim
" 使用 vim-claude 插件
Plug 'anthropic/vim-claude'

" 配置快捷键
nnoremap <leader>c :Claude<CR>
```

---

## 📝 最佳实践

### 1. 使用版本控制管理配置

```bash
cd ~/.claude
git init
git add .
git commit -m "Initial Claude config"
```

### 2. 备份重要配置

```bash
# 备份
tar -czf claude-config-backup.tar.gz ~/.claude

# 恢复
tar -xzf claude-config-backup.tar.gz -C ~/
```

### 3. 定期更新

```bash
# 检查更新
claude --check-update

# 自动更新
claude --update
```

---

## 🎯 下一步

环境搭建完成！现在你可以：

1. **阅读教程**：[01-项目概述](./01-project-overview.md)
2. **尝试示例**：运行 `claude` 开始对话
3. **配置项目**：创建 `.claude/CLAUDE.md`
4. **安装插件**：探索可用插件

祝你使用愉快！🚀
