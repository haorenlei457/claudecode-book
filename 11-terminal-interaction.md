# 11 - 终端交互

## 📋 模块介绍

终端交互是 Claude Code 与命令行环境交互的核心能力。本章将讲解终端操作和脚本执行。

---

## 🟢 入门级：终端操作基础

### 基本命令执行

```bash
# 运行命令
claude> 运行 npm install

# 查看文件
claude> 列出当前目录文件

# 执行脚本
claude> 运行 tests.sh
```

---

## 🟡 中级：高级终端操作

### 命令链

```bash
# 管道操作
claude> 运行 cat logs.txt | grep ERROR

# 后台任务
claude> 后台运行 npm start

# 重定向
claude> 运行 npm test > test-results.txt
```

---

## 🔴 专家级：终端交互深度剖析

### 终端管理器

```typescript
class TerminalManager {
  async execute(
    command: string,
    options: ExecOptions = {}
  ): Promise<ExecResult> {
    const { stdout, stderr } = await exec(command, {
      ...options,
      cwd: options.cwd || process.cwd()
    });

    return {
      stdout,
      stderr,
      exitCode: 0
    };
  }

  async executeInteractive(
    command: string
  ): Promise<void> {
    const pty = spawn('bash', ['-c', command], {
      stdio: 'inherit'
    });

    await new Promise(resolve => pty.on('close', resolve));
  }
}
```

---

## ✅ 章节总结

### 入门级要点
- ✅ 掌握基本命令执行

### 中级要点
- ✅ 掌握命令链操作

### 专家级要点
- ✅ 深入终端管理机制

---

**下一步：** 学习 [12 - 安全机制](./12-security.md) 🚀
