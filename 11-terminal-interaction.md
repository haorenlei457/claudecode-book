# 11 - 终端交互

## 📋 模块介绍

终端交互是 Claude Code 与操作系统对话的桥梁。通过强大的终端能力，Claude Code 可以执行命令、运行脚本、管理系统任务。本章将深入讲解终端操作的高级用法。

---

## 🟢 入门级：终端操作基础

### 基本命令执行

```bash
# 运行简单命令
claude> 运行 npm install
claude> 执行 python script.py

# 查看目录
claude> 列出当前目录
claude> 显示文件列表

# 查看文件
claude> 查看 README.md
claude> 显示 package.json 内容

# 文件操作
claude> 创建目录 src/components
claude> 复制文件 config.example.json config.json
claude> 移动文件 old.js archive/old.js
```

### 命令执行模式

Claude Code 支持多种命令执行模式：

| 模式 | 说明 | 示例 |
|------|------|------|
| **同步** | 等待命令完成 | `npm test` |
| **异步** | 后台运行 | `npm start &` |
| **交互** | 保留终端交互 | `git rebase -i` |
| **管道** | 命令链 | `cat file | grep pattern` |

---

## 🟡 中级：高级终端操作

### 命令链与管道

```bash
# 管道操作
claude> 运行 cat logs.txt | grep ERROR | head -20

# 多重管道
claude> 运行 ps aux | grep node | awk '{print $2}' | xargs kill

# 重定向输出
claude> 运行 npm test > test-results.txt 2>&1

# 命令组合
claude> 运行 npm run build && npm run deploy

# 条件执行
claude> 运行 git diff --quiet || echo "有未提交的变更"
```

### 环境变量管理

```bash
# 查看环境变量
claude> 显示环境变量 PATH
claude> 列出所有环境变量

# 设置环境变量
claude> 设置 NODE_ENV=production
claude> 添加 PATH: /usr/local/bin

# 使用环境变量
claude> 运行 echo $HOME
claude> 运行 echo 当前用户: $USER
```

### 后台任务管理

```bash
# 后台运行
claude> 后台运行 npm run watch

# 查看任务
claude> 显示后台任务

# 前台恢复
claude> 前台恢复任务 1

# 停止任务
claude> 停止后台任务 1
```

---

## 🔴 专家级：终端交互深度剖析

### 终端管理器实现

```typescript
class TerminalManager {
  private processes: Map<string, ChildProcess> = new Map();
  private outputBuffers: Map<string, string[]> = new Map();

  async execute(
    command: string,
    options: TerminalOptions = {}
  ): Promise<TerminalResult> {
    const { cwd = process.cwd(), env = process.env } = options;

    return new Promise((resolve, reject) => {
      const child = spawn('bash', ['-c', command], {
        cwd,
        env: { ...env, ...options.env },
        stdio: ['pipe', 'pipe', 'pipe']
      });

      let stdout = '';
      let stderr = '';

      child.stdout.on('data', (data) => {
        stdout += data.toString();
        if (options.onOutput) {
          options.onOutput(data.toString());
        }
      });

      child.stderr.on('data', (data) => {
        stderr += data.toString();
      });

      child.on('close', (code) => {
        if (code === 0 || options.ignoreExitCode) {
          resolve({
            stdout: stdout.trim(),
            stderr: stderr.trim(),
            exitCode: code || 0
          });
        } else {
          reject(new Error(
            `Command failed with exit code ${code}:\n${stderr}`
          ));
        }
      });

      child.on('error', (error) => {
        reject(error);
      });
    });
  }

  async executeInteractive(command: string): Promise<void> {
    const child = spawn('bash', ['-c', command], {
      stdio: 'inherit'
    });

    return new Promise((resolve, reject) => {
      child.on('close', (code) => {
        if (code === 0) {
          resolve();
        } else {
          reject(new Error(`Command exited with code ${code}`));
        }
      });
    });
  }
}
```

### 命令解析器

```typescript
class CommandParser {
  parse(input: string): ParsedCommand {
    // 解析命令字符串
    const tokens = this.tokenize(input);

    // 识别管道
    const pipes = this.splitByPipe(tokens);

    // 解析重定向
    const { command, redirects } = this.parseRedirects(pipes[0]);

    // 解析环境变量
    const { env, args } = this.parseEnvVars(command);

    return {
      command: args[0],
      args: args.slice(1),
      env,
      redirects,
      pipes: pipes.slice(1).map(p => this.parseSingleCommand(p))
    };
  }

  private tokenize(input: string): string[] {
    const tokens: string[] = [];
    let current = '';
    let inQuote = false;
    let quoteChar = '';

    for (const char of input) {
      if (char === '"' || char === "'") {
        if (!inQuote) {
          inQuote = true;
          quoteChar = char;
        } else if (char === quoteChar) {
          inQuote = false;
          quoteChar = '';
        } else {
          current += char;
        }
      } else if (char === ' ' && !inQuote) {
        if (current) {
          tokens.push(current);
          current = '';
        }
      } else {
        current += char;
      }
    }

    if (current) {
      tokens.push(current);
    }

    return tokens;
  }
}
```

---

## ✅ 章节总结

### 入门级要点
- ✅ 掌握基本命令执行
- ✅ 理解不同执行模式

### 中级要点
- ✅ 掌握命令链和管道
- ✅ 理解环境变量管理
- ✅ 学会后台任务管理

### 专家级要点
- ✅ 深入终端管理实现
- ✅ 掌握命令解析

---

**下一步：** 学习 [12 - 安全机制](./12-security.md) 🚀
