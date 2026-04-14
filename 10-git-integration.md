# 10 - Git集成

## 📋 模块介绍

Git集成让 Claude Code 能够直接操作Git仓库，包括提交、分支、PR管理等。本章将讲解Git集成的使用。

---

## 🟢 入门级：Git操作基础

### 基本Git命令

```bash
# 查看状态
claude> 查看git状态

# 创建提交
claude> /commit 修复登录bug

# 创建分支
claude> 创建新分支 feature/user-auth

# 查看历史
claude> 查看最近的提交
```

---

## 🟡 中级：Git工作流

### 提交流程

```markdown
1. 分析变更
   - 查看git diff
   - 识别修改的文件

2. 生成消息
   - 遵循Conventional Commits
   - 包含必要的细节

3. 执行提交
   - git add
   - git commit
```

### PR管理

```bash
# 创建PR
claude> 创建PR，标题是"添加用户认证"

# 审查PR
claude> 审查PR #123

# 合并PR
claude> 合并PR #123
```

---

## 🔴 专家级：Git集成深度剖析

### Git操作封装

```typescript
class GitOperations {
  async status(): Promise<GitStatus> {
    const result = await exec('git status --porcelain');
    return this.parseStatus(result.stdout);
  }

  async commit(message: string): Promise<void> {
    await exec('git add .');
    await exec(`git commit -m "${message}"`);
  }

  async createBranch(name: string): Promise<void> {
    await exec(`git checkout -b ${name}`);
  }

  private parseStatus(output: string): GitStatus {
    const lines = output.split('\n');
    return {
      modified: lines.filter(l => l.startsWith(' M')).length,
      added: lines.filter(l => l.startsWith('A ')).length,
      deleted: lines.filter(l => l.startsWith(' D')).length
    };
  }
}
```

---

## ✅ 章节总结

### 入门级要点
- ✅ 掌握基本Git命令
- ✅ 理解提交流程

### 中级要点
- ✅ 掌握PR管理
- ✅ 理解Git工作流

### 专家级要点
- ✅ 深入Git操作封装

---

**下一步：** 学习 [11 - 终端交互](./11-terminal-interaction.md) 🚀
