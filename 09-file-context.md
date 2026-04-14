# 09 - 文件操作与上下文管理

## 📋 模块介绍

文件操作和上下文管理是 Claude Code 的核心能力，让它能够理解和管理整个项目。本章将讲解文件系统和上下文管理。

---

## 🟢 入门级：文件操作基础

### 基本文件操作

```bash
# 读取文件
claude> 读取 package.json

# 写入文件
claude> 创建 README.md，内容是...

# 编辑文件
claude> 在 index.js 中添加日志

# 删除文件
claude> 删除 temp.py
```

### 上下文自动加载

```
项目/
├── CLAUDE.md          # 项目级上下文
├── src/
│   ├── CLAUDE.md      # 子项目上下文
│   └── main.js
└── tests/
    └── CLAUDE.md      # 测试上下文
```

---

## 🟡 中级：上下文管理

### CLAUDE.md 结构

```markdown
# 项目名称

## 项目类型
Python Web应用

## 技术栈
- FastAPI
- SQLAlchemy
- PostgreSQL

## 编码规范
- PEP 8
- 类型注解
- 文档字符串

## 常见任务
1. 运行测试：pytest
2. 启动服务：uvicorn main:app
3. 数据库迁移：alembic upgrade head
```

### 上下文继承

```
全局配置 (~/.claude/CLAUDE.md)
  ↑
项目配置 (./CLAUDE.md)
  ↑
子项目配置 (src/CLAUDE.md)
```

---

## 🔴 专家级：上下文管理深度剖析

### 上下文构建器

```typescript
class ContextBuilder {
  async buildContext(cwd: string): Promise<ProjectContext> {
    const context: ProjectContext = {
      files: [],
      structure: {},
      memory: {},
      metadata: {}
    };

    // 1. 扫描文件
    context.files = await this.scanFiles(cwd);

    // 2. 构建结构树
    context.structure = this.buildTree(context.files);

    // 3. 加载记忆
    context.memory = await this.loadMemory(cwd);

    // 4. 分析依赖
    context.metadata.dependencies = await this.analyzeDependencies(context);

    return context;
  }

  private async loadMemory(dir: string): Promise<Memory> {
    const memory: Memory = {};

    // 向上查找CLAUDE.md
    let current = dir;
    while (current !== '/') {
      const claudeFile = path.join(current, 'CLAUDE.md');
      if (await fs.pathExists(claudeFile)) {
        const content = await fs.readFile(claudeFile, 'utf-8');
        memory[current] = content;
      }
      current = path.dirname(current);
    }

    return memory;
  }
}
```

---

## ✅ 章节总结

### 入门级要点
- ✅ 掌握基本文件操作
- ✅ 理解自动上下文加载

### 中级要点
- ✅ 掌握CLAUDE.md结构
- ✅ 理解上下文继承

### 专家级要点
- ✅ 深入上下文构建机制

---

**下一步：** 学习 [10 - Git集成](./10-git-integration.md) 🚀
