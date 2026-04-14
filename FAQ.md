# ❓ Claude Code 常见问题解答

## 📚 快速导航

- [安装与配置](#安装与配置)
- [基本使用](#基本使用)
- [插件与扩展](#插件与扩展)
- [命令系统](#命令系统)
- [Git集成](#git集成)
- [性能与优化](#性能与优化)
- [故障排查](#故障排查)

---

## 🔧 安装与配置

### Q1: Claude Code 支持哪些操作系统？

**A:**
- ✅ macOS 10.15+
- ✅ Ubuntu 18.04+ / Debian 10+
- ✅ Windows 10+
- ✅ Linux 发行版（通过手动安装）

详细安装步骤请查看 [环境搭建指南](./SETUP_GUIDE.md)。

---

### Q2: 如何验证 Claude Code 是否安装成功？

**A:** 运行以下命令：
```bash
claude --version
claude --ping
```

如果显示版本号和 "✓ Connected"，说明安装成功。

---

### Q3: API 密钥如何配置？

**A:** 有三种方法：

**方法1：环境变量**
```bash
export ANTHROPIC_API_KEY="sk-ant-xxxxx"
```

**方法2：配置文件**
```bash
mkdir -p ~/.config/claude
echo '{"apiKey": "sk-ant-xxxxx"}' > ~/.config/claude/config.json
```

**方法3：运行时输入**
```bash
claude --prompt-for-key
```

---

### Q4: 如何卸载 Claude Code？

**A:**
```bash
# macOS/Linux
brew uninstall claude-code

# 或手动删除
rm /usr/local/bin/claude
rm -rf ~/.claude
```

详细步骤请查看 [环境搭建指南](./SETUP_GUIDE.md)。

---

## 🚀 基本使用

### Q5: Claude Code 和 ChatGPT 有什么区别？

**A:**

| 特性 | Claude Code | ChatGPT |
|------|-------------|---------|
| 运行环境 | 终端命令行 | Web界面 |
| 代码理解 | 深度理解项目结构 | 通用理解 |
| Git集成 | 原生集成 | 需要插件 |
| 文件操作 | 直接操作文件系统 | 需要复制粘贴 |
| 离线使用 | 部分功能 | 完全在线 |

Claude Code 专为开发者设计，能直接操作文件和Git。

---

### Q6: 如何让 Claude Code 理解我的项目？

**A:** 创建 `CLAUDE.md` 文件：

```markdown
# 项目名称

## 技术栈
- React + TypeScript
- Node.js + Express
- PostgreSQL

## 编码规范
- 使用 ESLint + Prettier
- 函数组件优先
- 所有函数必须有类型注解

## Git 工作流
- feature/* 分支
- Conventional Commits
- PR需要审查
```

更多信息请查看 [09-文件操作与上下文管理](./09-file-context.md)。

---

### Q7: 如何查看 Claude Code 的历史对话？

**A:** 对话历史保存在 `~/.claude/history/` 目录：
```bash
ls ~/.claude/history/
# 显示历史会话列表
```

---

## 🔌 插件与扩展

### Q8: 如何安装插件？

**A:** 有三种方法：

**方法1：命令安装**
```bash
claude> /plugin install code-review
```

**方法2：手动安装**
```bash
git clone https://github.com/anthropics/claude-code.git
cd claude-code/plugins/code-review
cp -r .claude ~/.claude/
```

**方法3：项目级安装**
```bash
cp -r plugins/my-plugin .claude/
```

详细说明请查看 [02-插件系统](./02-plugin-system.md)。

---

### Q9: 如何创建自己的插件？

**A:** 按照[插件系统教程](./02-plugin-system.md)的步骤：

1. 创建插件目录
2. 编写 `plugin.json`
3. 添加命令/代理/技能
4. 测试和调试

完整教程请查看 [02-插件系统](./02-plugin-system.md)。

---

### Q10: 插件不工作怎么办？

**A:** 排查步骤：

1. 检查插件是否已加载
```bash
claude> /plugins list
```

2. 检查插件元数据
```bash
cat .claude/plugins/your-plugin/.claude-plugin/plugin.json
```

3. 查看错误日志
```bash
claude> /logs
```

4. 重新加载插件
```bash
claude> /plugin reload your-plugin
```

---

## 💬 命令系统

### Q11: 如何创建自定义命令？

**A:** 创建命令文件：

```markdown
# .claude/commands/my-cmd.md
---
name: "my-cmd"
description: "My custom command"
---

命令内容...
```

详细步骤请查看 [03-命令系统](./03-command-system.md)。

---

### Q12: 如何查看所有可用命令？

**A:**
```bash
claude> /help
# 或
claude> /commands list
```

---

### Q13: 命令参数如何传递？

**A:**

**位置参数：**
```bash
claude> /my-cmd arg1 arg2 arg3
```

**命名参数：**
```bash
claude> /my-cmd --option1 value1 --option2 value2
```

**在命令中访问：**
```markdown
位置参数：{{args.0}}, {{args.1}}
命名参数：{{options.option1}}, {{options.option2}}
```

---

## 🤝 代理系统

### Q14: 代理和技能有什么区别？

**A:**

| 特性 | 代理 (Agent) | 技能 (Skill) |
|------|-------------|-------------|
| 触发方式 | 自动委派或手动 | 自动触发 |
| 智能程度 | 高（AI决策） | 中（预设规则） |
| 复杂度 | 高 | 低 |
| 用途 | 复杂任务 | 重复性任务 |

详细说明请查看 [04-代理系统](./04-agent-system.md) 和 [05-技能系统](./05-skill-system.md)。

---

### Q15: 如何让多个代理协作？

**A:** 在对话中自然描述，Claude会自动协调：

```bash
claude> 开发用户认证功能
# Claude会自动：
# 1. @code-architect 设计架构
# 2. @implementation-agent 实现代码
# 3. @test-engineer 编写测试
# 4. @code-reviewer 审查代码
```

---

## 🔗 Git集成

### Q16: 如何让 Claude Code 自动生成规范的提交信息？

**A:** 使用智能提交：

```bash
claude> /commit

# Claude会自动：
# 1. 分析变更
# 2. 运行检查
# 3. 生成Conventional Commits格式的提交信息
# 4. 执行提交
```

详细说明请查看 [10-Git集成](./10-git-integration.md)。

---

### Q17: 如何创建和管理 PR？

**A:**

**创建 PR：**
```bash
claude> /pr create

# Claude会自动：
# 1. 推送分支
# 2. 生成PR描述
# 3. 创建PR
```

**审查 PR：**
```bash
claude> /pr review #123
```

**合并 PR：**
```bash
claude> /pr merge #123
```

---

## ⚡ 性能与优化

### Q18: Claude Code 运行缓慢怎么办？

**A:** 优化建议：

1. **减少上下文大小**
```bash
# 在 CLAUDE.md 中配置
```
```json
{
  "memory": {
    "maxSize": 500
  }
}
```

2. **禁用不必要的插件**
```bash
claude> /plugin disable plugin-name
```

3. **使用缓存**
```bash
# 配置缓存大小
```
```json
{
  "cache": {
    "enabled": true,
    "maxSize": "1GB"
  }
}
```

---

### Q19: 如何减少 Token 使用？

**A:**

1. **限制回复长度**
```json
{
  "maxTokens": 2000
}
```

2. **使用更简洁的模型**
```json
{
  "model": "claude-haiku-4"
}
```

3. **精简上下文**
- 只包含必要的文件
- 定期清理历史

---

## 🔧 故障排查

### Q20: 遇到 "Permission denied" 错误？

**A:** 检查权限设置：

```bash
# 检查文件权限
ls -la ~/.claude/

# 修复权限
chmod +x ~/.claude/hooks/*.sh
```

或在配置中调整：
```json
{
  "permissions": {
    "file:write": ["./**"],
    "bash:run": true
  }
}
```

---

### Q21: 插件加载失败？

**A:** 排查步骤：

1. 检查插件语法
```bash
# 验证 JSON 格式
cat plugin.json | jq .
```

2. 查看详细错误
```bash
claude> /plugin validate your-plugin
```

3. 检查依赖
```bash
cd ~/.claude/plugins/your-plugin
npm install
```

---

### Q22: 网络连接问题？

**A:** 检查：

```bash
# 测试网络
ping anthropic.com

# 检查代理
echo $HTTP_PROXY
echo $HTTPS_PROXY

# 测试 API 连接
claude --ping
```

---

### Q23: 配置文件冲突？

**A:** 配置优先级（从高到低）：

1. 项目配置 `./.claude/settings.json`
2. 全局配置 `~/.claude/settings.json`
3. 默认配置

使用以下命令查看最终配置：
```bash
claude> /config show
```

---

## 🎓 进阶问题

### Q24: 如何贡献到 Claude Code？

**A:**

1. Fork 仓库
```bash
git clone https://github.com/your-username/claude-code.git
```

2. 创建分支
```bash
git checkout -b feature/your-feature
```

3. 提交更改
```bash
git add .
git commit -m "feat: add your feature"
```

4. 推送并创建 PR
```bash
git push origin feature/your-feature
```

详细指南请查看官方贡献文档。

---

### Q25: 如何调试 Claude Code？

**A:**

1. **启用调试模式**
```bash
claude --debug
```

2. **查看日志**
```bash
claude> /logs
tail -f ~/.claude/logs/debug.log
```

3. **启用详细输出**
```json
{
  "logging": {
    "level": "debug"
  }
}
```

---

### Q26: 如何集成到 IDE？

**A:** 支持：

- **VS Code**: 安装扩展
- **JetBrains**: 安装插件
- **Vim**: 使用插件

详细步骤请查看 [环境搭建指南](./SETUP_GUIDE.md) 的 IDE 集成部分。

---

## 🆘 获取更多帮助

如果以上 FAQ 没有解决你的问题：

1. **查看完整文档**
   - [环境搭建指南](./SETUP_GUIDE.md)
   - [学习路径](./LEARNING_PATH.md)
   - [实战项目](./PRACTICE_PROJECTS.md)

2. **社区支持**
   - [Discord 社区](https://anthropic.com/discord)
   - [GitHub Issues](https://github.com/anthropics/claude-code/issues)

3. **官方文档**
   - [Claude Code 官方文档](https://code.claude.com/docs/en/overview)
   - [API 文档](https://docs.claude.com/en/api/claude-code)

---

**提示**：使用浏览器搜索功能（Ctrl+F / Cmd+F）快速查找问题！🔍
