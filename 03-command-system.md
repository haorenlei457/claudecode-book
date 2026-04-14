# 03 - 命令系统（Slash Commands）

## 📋 模块介绍

命令系统是 Claude Code 与用户交互的核心界面，通过斜杠命令（`/command`）提供快速、可复用的功能。本章将全面讲解命令的设计、开发和使用。

---

## 🟢 入门级：命令基础

### 什么是命令？

命令（Command）是以 `/` 开头的快捷指令，用于触发特定功能。

**类比理解**：
- 📱 类似微信/Slack 的 `/commands`
- 🎮 类似游戏的快捷键
- 🔧 类似 CLI 的 alias

### 命令 vs 自然语言

```bash
# 自然语言方式
claude> 帮我创建一个Git提交，消息是"修复登录bug"

# 命令方式
claude> /commit 修复登录bug
```

**命令的优势**：
- ⚡ 更快 - 一行搞定
- 🎯 更准 - 不需要AI理解意图
- 🔁 可复用 - 多次使用
- 📝 可记录 - 易于追踪

### 常用命令示例

| 命令 | 功能 | 示例 |
|------|------|------|
| `/commit` | 创建Git提交 | `/commit 修复登录问题` |
| `/review` | 代码审查 | `/review main.py` |
| `/optimize` | 代码优化 | `/optimize` |
| `/test` | 运行测试 | `/test --watch` |
| `/docs` | 生成文档 | `/docs API.md` |
| `/deploy` | 部署应用 | `/deploy staging` |

### 命令的组成部分

```
/command [参数] [选项]

  ↓      ↓       ↓
命令名  必选参数 可选参数
```

**示例**：

```bash
# 简单命令
/commit

# 带参数
/commit 修复bug

# 带选项
/commit --amend --no-verify

# 复杂命令
/deploy --env production --skip-tests --tag v1.0.0
```

### 如何使用命令？

#### 1. 查看可用命令

```bash
$ claude
claude> /help
可用命令：
  /commit    - 创建Git提交
  /review    - 代码审查
  /optimize  - 优化代码
  /test      - 运行测试
  ...
```

#### 2. 查看命令帮助

```bash
claude> /help commit
命令：/commit
描述：创建符合规范的Git提交消息

用法：
  /commit [消息]

示例：
  /commit 修复登录bug
  /commit feat: 添加用户注册功能

选项：
  --amend    修改最后一次提交
  --no-verify 跳过Git hooks
```

#### 3. 执行命令

```bash
claude> /commit 修复登录验证问题

✅ 已创建提交：
commit abc123def
Author: Your Name <you@example.com>
Date:   Mon Apr 14 12:00:00 2026

    fix: 修复登录验证问题

    - 修复了token验证逻辑
    - 添加了错误处理
    - 更新了单元测试
```

---

## 🟡 中级：命令开发与设计

### 命令定义格式

命令使用 Markdown + YAML frontmatter 定义：

```markdown
---
name: "commit"
description: "Create a Git commit with message"
alias: ["/git-commit", "/ci"]
options:
  - name: "amend"
    type: "boolean"
    description: "Amend the last commit"
  - name: "message"
    type: "string"
    required: true
    description: "Commit message"
---

你是Git提交专家。请按照以下步骤操作：

1. **分析变更**
   - 检查git状态
   - 查看暂存文件
   - 分析代码差异

2. **生成消息**
   - 遵循Conventional Commits
   - 包含必要的细节
   - 符合项目规范

3. **执行提交**
   {{git:commit --message="{{message}}" {{amend}}}}

4. **验证结果**
   - 确认提交成功
   - 显示提交信息

{{args}}
{{options}}
```

### Frontmatter 字段详解

| 字段 | 类型 | 必需 | 说明 |
|------|------|------|------|
| `name` | string | ✅ | 命令唯一标识 |
| `description` | string | ✅ | 命令描述 |
| `alias` | string[] | ❌ | 命令别名 |
| `options` | Option[] | ❌ | 命令选项 |
| `permissions` | string[] | ❌ | 需要的权限 |
| `category` | string | ❌ | 命令分类 |

### 命令参数处理

#### 1. 位置参数

```markdown
---
name: "greet"
description: "Greet someone"
---

你好，{{args.0}}！

{{#if args.1}}
今天天气不错，{{args.1}}
{{/if}}
```

使用：
```bash
claude> /greet 世界
你好，世界！

claude> /greet 世界 适合写代码
你好，世界！
今天天气不错，适合写代码
```

#### 2. 命名参数

```markdown
---
name: "deploy"
description: "Deploy application"
options:
  - name: "env"
    type: "string"
    default: "production"
  - name: "skip-tests"
    type: "boolean"
    default: false
---

部署到 {{options.env}} 环境...

{{#unless options.skip-tests}}
运行测试...
{{/unless}}

开始部署...
```

使用：
```bash
claude> /deploy --env staging --skip-tests
部署到 staging 环境...

开始部署...
```

### 命令模板语法

#### 1. 变量插值

```markdown
用户输入：{{args.0}}
选项值：{{options.verbose}}
环境变量：{{env.HOME}}
```

#### 2. 条件判断

```markdown
{{#if options.verbose}}
详细模式已启用
{{/if}}

{{#unless options.skip-tests}}
运行测试...
{{/unless}}
```

#### 3. 循环

```markdown
{{#each args}}
处理文件: {{this}}
{{/each}}
```

#### 4. 工具调用

```markdown
{{file:read ./config.json}}

{{git:status}}

{{bash:run npm test}}
```

### 创建自定义命令

#### 步骤1：创建命令文件

```bash
mkdir -p .claude/commands
```

#### 步骤2：编写命令

创建 `.claude/commands/generate-report.md`：

```markdown
---
name: "generate-report"
description: "Generate project report"
alias: ["/report", "/stats"]
---

生成项目报告...

## 代码统计

{{bash:run find . -name "*.py" -type f | wc -l}} 个 Python 文件
{{bash:run find . -name "*.js" -type f | wc -l}} 个 JavaScript 文件

## Git 信息

当前分支：{{bash:run git branch --show-current}}
最近提交：{{bash:run git log -1 --pretty=format:"%h - %s (%cr)"}}

## 测试状态

{{bash:run npm test -- --passWithNoTests}}

## 报告生成时间
{{date}}
```

#### 步骤3：测试命令

```bash
$ claude
claude> /generate-report
生成项目报告...

## 代码统计
42 个 Python 文件
15 个 JavaScript 文件

## Git 信息
当前分支：main
最近提交：abc123 - 修复bug (2小时前)

## 测试状态
✅ 所有测试通过

## 报告生成时间
2026-04-14 12:00:00
```

### 命令权限管理

```markdown
---
name: "deploy-prod"
description: "Deploy to production"
permissions:
  - "git:write"
  - "file:read"
  - "bash:run"
---

部署到生产环境...

⚠️  警告：这将影响生产环境！

请确认：
1. 代码已测试
2. 回滚方案已准备
3. 团队已通知

{{#confirm}}
继续部署？
{{/confirm}}
```

---

## 🔴 专家级：命令系统深度剖析

### 命令解析器实现

```typescript
class CommandParser {
  private commands: Map<string, Command>;

  parse(input: string): ParseResult {
    // 1. 检查是否是命令
    if (!input.startsWith('/')) {
      return { type: 'chat', content: input };
    }

    // 2. 提取命令部分
    const parts = input.slice(1).split(/\s+/);
    const commandName = parts[0];
    const rawArgs = parts.slice(1);

    // 3. 查找命令定义
    const command = this.findCommand(commandName);
    if (!command) {
      return { type: 'error', message: 'Unknown command' };
    }

    // 4. 解析参数和选项
    const { args, options } = this.parseArguments(rawArgs, command);

    return {
      type: 'command',
      command,
      args,
      options
    };
  }

  private parseArguments(
    raw: string[],
    command: Command
  ): { args: string[], options: Record<string, any> } {
    const args: string[] = [];
    const options: Record<string, any> = {};

    for (let i = 0; i < raw.length; i++) {
      const token = raw[i];

      // 选项
      if (token.startsWith('--')) {
        const key = token.slice(2);
        const value = raw[i + 1];

        if (command.options.find(o => o.name === key)) {
          options[key] = value || true;
          if (value) i++; // 跳过值
        }
      }
      // 短选项
      else if (token.startsWith('-')) {
        const key = token.slice(1);
        const option = command.options.find(o =>
          o.short && o.short === key
        );

        if (option) {
          const value = raw[i + 1];
          options[option.name] = value || true;
          if (value && option.type !== 'boolean') i++;
        }
      }
      // 位置参数
      else {
        args.push(token);
      }
    }

    return { args, options };
  }
}
```

### 命令执行引擎

```typescript
class CommandExecutor {
  private parser: CommandParser;
  private templateEngine: TemplateEngine;

  async execute(parseResult: ParseResult): Promise<void> {
    switch (parseResult.type) {
      case 'command':
        await this.executeCommand(parseResult);
        break;

      case 'chat':
        await this.executeChat(parseResult);
        break;

      case 'error':
        this.showError(parseResult.message);
        break;
    }
  }

  private async executeCommand(result: CommandParseResult): Promise<void> {
    const { command, args, options } = result;

    // 1. 验证参数
    this.validateArguments(command, args, options);

    // 2. 准备上下文
    const context = await this.buildContext(command, args, options);

    // 3. 渲染模板
    const rendered = await this.templateEngine.render(
      command.template,
      context
    );

    // 4. 执行工具调用
    const toolCalls = this.extractToolCalls(rendered);
    for (const call of toolCalls) {
      await this.executeTool(call);
    }

    // 5. 返回结果
    this.output(rendered);
  }

  private async buildContext(
    command: Command,
    args: string[],
    options: Record<string, any>
  ): Promise<Context> {
    return {
      args,
      options,
      env: process.env,
      date: new Date().toISOString(),
      // 扩展上下文
      ...await this.loadExtensions(command)
    };
  }
}
```

### 模板引擎实现

```typescript
class TemplateEngine {
  async render(template: string, context: Context): Promise<string> {
    // 1. 解析模板
    const ast = this.parse(template);

    // 2. 渲染AST
    return this.renderAST(ast, context);
  }

  private renderAST(ast: ASTNode, context: Context): string {
    let result = '';

    for (const node of ast.nodes) {
      switch (node.type) {
        case 'text':
          result += node.content;
          break;

        case 'variable':
          result += this.evaluateVariable(node.path, context);
          break;

        case 'if':
          if (this.evaluateCondition(node.condition, context)) {
            result += this.renderAST(node.consequent, context);
          } else if (node.alternate) {
            result += this.renderAST(node.alternate, context);
          }
          break;

        case 'each':
          const items = this.evaluateVariable(node.collection, context);
          for (const item of items) {
            const itemContext = { ...context, [node.item]: item };
            result += this.renderAST(node.body, itemContext);
          }
          break;

        case 'tool-call':
          const result = await this.executeTool(node.tool, node.args, context);
          result += result;
          break;
      }
    }

    return result;
  }

  private evaluateVariable(path: string, context: Context): any {
    const parts = path.split('.');
    let value = context;

    for (const part of parts) {
      value = value?.[part];
    }

    return value;
  }
}
```

### 命令别名系统

```typescript
class AliasResolver {
  private aliases: Map<string, string>;

  resolve(input: string): string {
    // 1. 检查是否是别名
    const alias = this.aliases.get(input);
    if (alias) {
      return this.resolve(alias); // 递归解析
    }

    return input;
  }

  register(alias: string, command: string): void {
    this.aliases.set(alias, command);
  }

  // 示例别名
  registerCommonAliases(): void {
    this.register('/ci', '/commit');
    this.register('/ps', '/push');
    this.register('/pr', '/pull-request');
    this.register('/fix', '/commit --amend');
  }
}
```

### 命令历史管理

```typescript
class CommandHistory {
  private history: string[] = [];
  private index: number = -1;
  private maxSize: number = 1000;

  add(command: string): void {
    // 不重复添加
    if (this.history[this.history.length - 1] !== command) {
      this.history.push(command);
    }

    // 限制大小
    if (this.history.length > this.maxSize) {
      this.history.shift();
    }

    this.index = this.history.length;
  }

  getPrevious(): string | null {
    if (this.index > 0) {
      this.index--;
      return this.history[this.index];
    }
    return null;
  }

  getNext(): string | null {
    if (this.index < this.history.length - 1) {
      this.index++;
      return this.history[this.index];
    }
    return null;
  }

  search(query: string): string[] {
    return this.history.filter(cmd =>
      cmd.toLowerCase().includes(query.toLowerCase())
    );
  }
}
```

### 命令自动补全

```typescript
class CommandCompleter {
  async complete(partial: string): Promise<Completion[]> {
    // 1. 识别补全类型
    if (partial.startsWith('/')) {
      return this.completeCommand(partial);
    }

    return [];
  }

  private async completeCommand(partial: string): Promise<Completion[]> {
    const parts = partial.split(/\s+/);
    const commandName = parts[0];
    const context = parts.slice(1);

    // 补全命令名
    if (parts.length === 1) {
      return this.completeCommandName(commandName);
    }

    // 补全参数
    const command = this.findCommand(commandName);
    if (command) {
      return this.completeArguments(command, context);
    }

    return [];
  }

  private completeCommandName(partial: string): Completion[] {
    const commands = this.getAllCommands();

    return commands
      .filter(cmd => cmd.name.startsWith(partial))
      .map(cmd => ({
        type: 'command',
        value: cmd.name,
        description: cmd.description
      }));
  }

  private completeArguments(
    command: Command,
    context: string[]
  ): Completion[] {
    const completions: Completion[] = [];

    // 选项补全
    for (const option of command.options) {
      const flag = `--${option.name}`;
      if (context.includes(flag)) continue;

      completions.push({
        type: 'option',
        value: flag,
        description: option.description
      });

      if (option.short) {
        completions.push({
          type: 'option',
          value: `-${option.short}`,
          description: option.description
        });
      }
    }

    // 文件补全
    if (command.options.some(o => o.type === 'file')) {
      const files = await this.listFiles('.');
      completions.push(
        ...files.map(f => ({
          type: 'file',
          value: f,
          description: 'File'
        }))
      );
    }

    return completions;
  }
}
```

### 命令性能优化

```typescript
class CommandOptimizer {
  private cache: Map<string, CachedResult>;

  async execute(
    command: Command,
    args: string[],
    options: Record<string, any>
  ): Promise<void> {
    // 1. 检查缓存
    const cacheKey = this.computeCacheKey(command, args, options);
    const cached = this.cache.get(cacheKey);

    if (cached && !this.isExpired(cached)) {
      this.output(cached.result);
      return;
    }

    // 2. 执行命令
    const result = await this.executeUncached(command, args, options);

    // 3. 缓存结果
    this.cache.set(cacheKey, {
      result,
      timestamp: Date.now(),
      ttl: command.cacheTTL || 60000
    });

    this.output(result);
  }

  private computeCacheKey(
    command: Command,
    args: string[],
    options: Record<string, any>
  ): string {
    const data = {
      command: command.name,
      args,
      options,
      // 包含相关文件哈希
      files: this.getRelevantFileHashes(command)
    };

    return hash(JSON.stringify(data));
  }

  private isExpired(cached: CachedResult): boolean {
    return Date.now() - cached.timestamp > cached.ttl;
  }
}
```

### 命令安全性

```typescript
class CommandSecurity {
  validate(command: Command, context: Context): SecurityResult {
    const issues: SecurityIssue[] = [];

    // 1. 检查权限
    for (const permission of command.permissions) {
      if (!this.hasPermission(permission)) {
        issues.push({
          type: 'permission_denied',
          permission
        });
      }
    }

    // 2. 检查危险操作
    if (this.isDangerousCommand(command)) {
      issues.push({
        type: 'dangerous_operation',
        command: command.name
      });
    }

    // 3. 检查参数注入
    for (const arg of context.args) {
      if (this.detectInjection(arg)) {
        issues.push({
          type: 'injection_attempt',
          argument: arg
        });
      }
    }

    return {
      safe: issues.length === 0,
      issues
    };
  }

  private hasPermission(permission: string): boolean {
    // 检查用户权限
    return true;
  }

  private isDangerousCommand(command: Command): boolean {
    const dangerous = ['rm', 'deploy-prod', 'purge'];
    return dangerous.includes(command.name);
  }

  private detectInjection(input: string): boolean {
    const patterns = [
      /[;&|`$()]/,  // Shell注入
      /\.\.[/\\]/, // 路径遍历
      /<script/i,   // XSS
      /https?:\/\// // URL注入
    ];

    return patterns.some(pattern => pattern.test(input));
  }
}
```

---

## 📚 实战案例：开发测试命令

### 需求

- 📊 运行所有测试
- 🎯 运行特定测试
- 🔁 监听模式
- 📈 生成覆盖率报告

### 实现

#### 1. 命令定义

```markdown
---
name: "test"
description: "Run tests"
alias: ["/t", "/run-tests"]
options:
  - name: "watch"
    type: "boolean"
    short: "w"
    description: "Watch mode"
  - name: "coverage"
    type: "boolean"
    short: "c"
    description: "Generate coverage report"
  - name: "pattern"
    type: "string"
    short: "p"
    description: "Test pattern"
permissions:
  - "bash:run"
---

运行测试...

{{#if options.coverage}}
生成覆盖率报告...
{{bash:run npm test -- --coverage}}
{{/if}}

{{#if options.watch}}
启动监听模式...
{{bash:run npm test -- --watch}}
{{/if}}

{{#if options.pattern}}
运行匹配的测试: {{options.pattern}}
{{bash:run npm test -- --grep "{{options.pattern}}"}}
{{/if}}

{{#if (and (not options.coverage) (not options.watch) (not options.pattern))}}
运行所有测试...
{{bash:run npm test}}
{{/if}}
```

#### 2. 使用示例

```bash
# 运行所有测试
claude> /test

# 运行特定测试
claude> /test --pattern "login"

# 监听模式
claude> /test --watch

# 生成覆盖率
claude> /test --coverage

# 组合使用
claude> /test -w -c
```

---

## ✅ 章节总结

### 入门级要点
- ✅ 理解命令的概念
- ✅ 掌握基本使用方法
- ✅ 了解常用命令

### 中级要点
- ✅ 掌握命令定义格式
- ✅ 理解参数和选项
- ✅ 掌握模板语法
- ✅ 学会创建自定义命令

### 专家级要点
- ✅ 深入解析器实现
- ✅ 掌握执行引擎
- ✅ 理解模板引擎
- ✅ 掌握性能优化
- ✅ 了解安全机制

---

**下一步：** 学习 [04 - 代理系统](./04-agent-system.md) 🚀
