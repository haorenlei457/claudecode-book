# 09 - 文件操作与上下文管理

## 📋 模块介绍

文件操作和上下文管理是 Claude Code 理解项目的基础能力。通过智能的文件扫描和上下文构建，Claude Code 能够"读懂"整个项目，提供精准的代码建议。本章将深入讲解文件系统和上下文管理。

---

## 🟢 入门级：文件操作基础

### 基本文件操作

Claude Code 提供多种文件操作方式：

```bash
# 读取文件内容
claude> 读取 package.json 的内容
claude> 查看 src/main.py

# 写入文件
claude> 创建一个新文件 config.json，内容是 {...}
claude> 在 src/utils.js 中添加一个函数

# 编辑文件
claude> 修改 README.md，添加安装说明
claude> 在 index.html 的 head 中添加 meta 标签

# 删除文件
claude> 删除临时文件 temp.txt
claude> 移除旧的配置文件

# 搜索文件
claude> 搜索包含 "TODO" 的所有文件
claude> 查找所有测试文件
```

### 支持的文件类型

| 类型 | 扩展名 | 特殊处理 |
|------|--------|----------|
| 代码文件 | .js, .ts, .py, .go, .rs | 语法分析 |
| 配置文件 | .json, .yaml, .toml | 结构验证 |
| 文档 | .md, .txt, .rst | 全文检索 |
| 数据文件 | .csv, .xml | 格式解析 |
| 二进制 | .png, .jpg, .pdf | 元数据提取 |

### 文件操作安全

```bash
# Claude Code 会自动保护重要文件
⚠️  尝试写入 /etc/hosts
❌ 操作被拒绝：系统文件受保护

# 路径遍历防护
⚠️  路径包含 .. 
❌ 操作被拒绝：潜在的安全风险
```

---

## 🟡 中级：上下文管理

### 什么是上下文？

上下文（Context）是 Claude Code 对项目的**整体理解**，包括：

```
项目上下文
├── 文件结构
│   ├── 目录树
│   ├── 文件列表
│   └── 文件关系
├── 代码理解
│   ├── 模块依赖
│   ├── 函数调用
│   └── 类型信息
├── 项目配置
│   ├── 技术栈
│   ├── 编码规范
│   └── 工作流
└── 历史记录
    ├── 之前的对话
    ├── 修改记录
    └── 用户偏好
```

### CLAUDE.md 上下文系统

#### 1. 上下文层级

```
~/.claude/CLAUDE.md                    # 个人偏好
    ↓
./CLAUDE.md                           # 项目根配置
    ↓
./src/CLAUDE.md                       # 源代码规则
    ↓
./src/api/CLAUDE.md                   # API层规则
    ↓
./src/api/routes/CLAUDE.md            # 路由规则
```

**加载顺序**：从根到当前目录，逐步合并

#### 2. 上下文内容示例

**项目根 CLAUDE.md**：

```markdown
# MyProject

## 项目概述
这是一个现代化的Web应用

## 技术栈
- Frontend: React + TypeScript
- Backend: Node.js + Express
- Database: PostgreSQL
- ORM: Prisma

## 编码规范
- ESLint + Prettier
- Conventional Commits
- 测试覆盖率 > 80%

## Git 工作流
- main: 生产分支
- develop: 开发分支
- feature/*: 功能分支
```

**API层 CLAUDE.md**（src/api/CLAUDE.md）：

```markdown
# API 层配置

## 技术栈
- Express.js
- Zod (输入验证)
- Prisma (数据库)

## 目录结构
```
api/
├── controllers/    # 控制器
├── services/       # 业务逻辑
├── middleware/     # 中间件
├── routes/         # 路由定义
└── types/          # 类型定义
```

## 编码规范
- 控制器只处理HTTP相关逻辑
- 业务逻辑放在 services
- 统一错误处理
- 所有输入必须经过验证

## API 设计原则
1. RESTful 设计
2. 使用资源复数形式
3. HTTP 状态码规范
4. 统一响应格式

## 错误处理
```typescript
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input",
    "details": [...]
  }
}
```
```

### 上下文自动构建

Claude Code 如何构建上下文：

```typescript
interface ContextBuilder {
  build(cwd: string): Promise<ProjectContext>;
}

class DefaultContextBuilder implements ContextBuilder {
  async build(cwd: string): Promise<ProjectContext> {
    const context: ProjectContext = {
      files: [],
      structure: {},
      memory: {},
      metadata: {},
      dependencies: {},
      types: {}
    };

    // 1. 扫描文件系统
    context.files = await this.scanFiles(cwd);

    // 2. 构建目录结构
    context.structure = this.buildTree(context.files);

    // 3. 加载记忆文件
    context.memory = await this.loadMemory(cwd);

    // 4. 分析项目类型
    context.metadata.projectType = this.detectProjectType(context);

    // 5. 解析依赖关系
    context.dependencies = await this.analyzeDependencies(context);

    // 6. 提取类型信息
    context.types = await this.extractTypes(context);

    return context;
  }

  private async scanFiles(dir: string): Promise<FileInfo[]> {
    const files: FileInfo[] = [];
    const ignorePatterns = [
      '.git',
      'node_modules',
      'dist',
      'build',
      '.env',
      '*.log'
    ];

    const entries = await fs.readdir(dir, { withFileTypes: true });

    for (const entry of entries) {
      const fullPath = path.join(dir, entry.name);

      // 检查忽略模式
      if (this.shouldIgnore(entry.name, ignorePatterns)) {
        continue;
      }

      if (entry.isDirectory()) {
        // 递归扫描子目录
        const subFiles = await this.scanFiles(fullPath);
        files.push(...subFiles);
      } else {
        // 获取文件信息
        const stats = await fs.stat(fullPath);
        const content = await this.readFile(fullPath);

        files.push({
          path: fullPath,
          relativePath: path.relative(dir, fullPath),
          size: stats.size,
          modified: stats.mtime,
          extension: path.extname(entry.name),
          content: content.slice(0, 10000), // 限制内容大小
          hash: this.computeHash(content)
        });
      }
    }

    return files;
  }

  private async loadMemory(dir: string): Promise<MemoryFiles> {
    const memory: MemoryFiles = {};

    // 从当前目录向上查找 CLAUDE.md
    let currentDir = dir;
    while (currentDir !== path.dirname(currentDir)) {
      const memoryPath = path.join(currentDir, 'CLAUDE.md');

      if (await fs.pathExists(memoryPath)) {
        const content = await fs.readFile(memoryPath, 'utf-8');
        memory[currentDir] = content;
      }

      currentDir = path.dirname(currentDir);
    }

    return memory;
  }

  private detectProjectType(context: ProjectContext): string {
    const files = context.files.map(f => f.path);

    // 检查 package.json
    if (files.some(f => f.includes('package.json'))) {
      const packageJson = files.find(f => f.includes('package.json'));
      const content = JSON.parse(packageJson?.content || '{}');

      if (content.dependencies?.react) return 'react';
      if (content.dependencies?.vue) return 'vue';
      if (content.dependencies?.express) return 'express';
      return 'node';
    }

    // 检查 Python
    if (files.some(f => f.endsWith('requirements.txt'))) {
      return 'python';
    }
    if (files.some(f => f.endsWith('pyproject.toml'))) {
      return 'python';
    }

    // 检查 Go
    if (files.some(f => f.endsWith('go.mod'))) {
      return 'go';
    }

    // 检查 Rust
    if (files.some(f => f.endsWith('Cargo.toml'))) {
      return 'rust';
    }

    return 'unknown';
  }
}
```

---

## 🔴 专家级：上下文管理深度剖析

### 智能文件过滤

```typescript
class SmartFileFilter {
  private relevanceScores: Map<string, number> = new Map();

  async filter(
    files: FileInfo[],
    query: string,
    limit: number = 50
  ): Promise<FileInfo[]> {
    // 1. 计算每个文件的相关性
    for (const file of files) {
      const score = await this.calculateRelevance(file, query);
      this.relevanceScores.set(file.path, score);
    }

    // 2. 排序并返回前N个
    const sorted = files.sort((a, b) => {
      const scoreA = this.relevanceScores.get(a.path) || 0;
      const scoreB = this.relevanceScores.get(b.path) || 0;
      return scoreB - scoreA;
    });

    return sorted.slice(0, limit);
  }

  private async calculateRelevance(
    file: FileInfo,
    query: string
  ): Promise<number> {
    let score = 0;

    // 文件名匹配
    if (file.path.toLowerCase().includes(query.toLowerCase())) {
      score += 10;
    }

    // 内容匹配
    const contentMatches = (file.content.match(new RegExp(query, 'gi')) || []).length;
    score += contentMatches * 2;

    // 文件类型权重
    const typeWeights: Record<string, number> = {
      '.ts': 1.5,
      '.tsx': 1.5,
      '.js': 1.2,
      '.json': 1.0,
      '.md': 0.8
    };
    score *= typeWeights[file.extension] || 1;

    // 最近修改的文件加分
    const daysSinceModified = (Date.now() - file.modified.getTime()) / (1000 * 60 * 60 * 24);
    if (daysSinceModified < 7) {
      score *= 1.5;
    }

    return score;
  }
}
```

### 增量上下文更新

```typescript
class IncrementalContextUpdater {
  private context: ProjectContext;
  private fileHashes: Map<string, string> = new Map();

  async update(changedFiles: string[]): Promise<ProjectContext> {
    for (const filePath of changedFiles) {
      const newHash = await this.computeFileHash(filePath);
      const oldHash = this.fileHashes.get(filePath);

      if (oldHash !== newHash) {
        // 文件已修改，更新上下文
        await this.updateFileInContext(filePath);
        this.fileHashes.set(filePath, newHash);
      }
    }

    // 移除已删除的文件
    for (const [path, hash] of this.fileHashes.entries()) {
      if (!await fs.pathExists(path)) {
        this.removeFileFromContext(path);
        this.fileHashes.delete(path);
      }
    }

    return this.context;
  }

  private async updateFileInContext(filePath: string): Promise<void> {
    const content = await fs.readFile(filePath, 'utf-8');
    const stats = await fs.stat(filePath);

    const fileInfo: FileInfo = {
      path: filePath,
      content,
      size: stats.size,
      modified: stats.mtime,
      hash: this.computeHash(content)
    };

    // 更新或添加文件
    const existingIndex = this.context.files.findIndex(
      f => f.path === filePath
    );

    if (existingIndex >= 0) {
      this.context.files[existingIndex] = fileInfo;
    } else {
      this.context.files.push(fileInfo);
    }

    // 更新依赖关系
    await this.updateDependencies(fileInfo);
  }
}
```

### 代码语义分析

```typescript
class CodeAnalyzer {
  async analyze(file: FileInfo): Promise<CodeAnalysis> {
    const analysis: CodeAnalysis = {
      imports: [],
      exports: [],
      functions: [],
      classes: [],
      types: [],
      dependencies: []
    };

    switch (file.extension) {
      case '.ts':
      case '.tsx':
      case '.js':
        return this.analyzeJavaScript(file);
      case '.py':
        return this.analyzePython(file);
      case '.go':
        return this.analyzeGo(file);
      default:
        return analysis;
    }
  }

  private analyzeJavaScript(file: FileInfo): CodeAnalysis {
    const analysis: CodeAnalysis = {
      imports: [],
      exports: [],
      functions: [],
      classes: [],
      types: [],
      dependencies: []
    };

    // 解析 import 语句
    const importRegex = /import\s+(?:(\{[^}]+\})|(\*\s+as\s+\w+)|(\w+))\s+from\s+['"]([^'"]+)['"];?/g;
    let match;
    while ((match = importRegex.exec(file.content)) !== null) {
      analysis.imports.push({
        source: match[4],
        names: match[1] || match[2] || match[3]
      });
    }

    // 解析 export 语句
    const exportRegex = /export\s+(?:default\s+)?(?:const|let|var|function|class|interface)?\s*(\w+)/g;
    while ((match = exportRegex.exec(file.content)) !== null) {
      analysis.exports.push(match[1]);
    }

    // 解析函数
    const functionRegex = /(?:async\s+)?function\s+(\w+)|(?:const|let|var)\s+(\w+)\s*=\s*(?:async\s+)?\([^)]*\)\s*=>/g;
    while ((match = functionRegex.exec(file.content)) !== null) {
      analysis.functions.push(match[1] || match[2]);
    }

    // 解析类
    const classRegex = /class\s+(\w+)(?:\s+extends\s+(\w+))?/g;
    while ((match = classRegex.exec(file.content)) !== null) {
      analysis.classes.push({
        name: match[1],
        extends: match[2]
      });
    }

    return analysis;
  }
}
```

---

## 📚 实战案例：智能代码搜索工具

### 需求
- 🔍 语义化代码搜索
- 📊 智能文件推荐
- 🔗 依赖关系分析
- 📈 代码复杂度评估

### 实现

```typescript
class SmartCodeSearch {
  private context: ProjectContext;

  async search(query: string): Promise<SearchResult[]> {
    // 1. 解析查询意图
    const intent = this.parseIntent(query);

    // 2. 根据意图选择搜索策略
    switch (intent.type) {
      case 'function':
        return this.searchByFunction(intent.value);
      case 'class':
        return this.searchByClass(intent.value);
      case 'pattern':
        return this.searchByPattern(intent.value);
      default:
        return this.semanticSearch(query);
    }
  }

  private parseIntent(query: string): QueryIntent {
    // 函数搜索
    const funcMatch = query.match(/(?:find|search for)?\s*(?:function|method)\s+(\w+)/i);
    if (funcMatch) {
      return { type: 'function', value: funcMatch[1] };
    }

    // 类搜索
    const classMatch = query.match(/(?:find|search for)?\s*class\s+(\w+)/i);
    if (classMatch) {
      return { type: 'class', value: classMatch[1] };
    }

    // 模式搜索
    const patternMatch = query.match(/(?:find|search for)?\s*pattern\s+(.+)/i);
    if (patternMatch) {
      return { type: 'pattern', value: patternMatch[1] };
    }

    return { type: 'semantic', value: query };
  }

  private semanticSearch(query: string): SearchResult[] {
    const results: SearchResult[] = [];

    for (const file of this.context.files) {
      // 计算语义相似度
      const similarity = this.calculateSimilarity(
        query,
        file.content
      );

      if (similarity > 0.3) {
        results.push({
          file: file.path,
          relevance: similarity,
          snippets: this.extractRelevantSnippets(query, file.content)
        });
      }
    }

    return results.sort((a, b) => b.relevance - a.relevance);
  }

  private calculateSimilarity(query: string, content: string): number {
    // 使用简单的 TF-IDF 计算相似度
    const queryTerms = this.tokenize(query);
    const contentTerms = this.tokenize(content);

    const querySet = new Set(queryTerms);
    const contentSet = new Set(contentTerms);

    const intersection = new Set(
      [...querySet].filter(x => contentSet.has(x))
    );

    return intersection.size / querySet.size;
  }

  private tokenize(text: string): string[] {
    return text
      .toLowerCase()
      .replace(/[^\w\s]/g, ' ')
      .split(/\s+/)
      .filter(word => word.length > 2);
  }
}
```

---

## ✅ 章节总结

### 入门级要点
- ✅ 掌握基本文件操作
- ✅ 理解支持的文件类型
- ✅ 了解文件操作安全

### 中级要点
- ✅ 掌握上下文概念
- ✅ 理解CLAUDE.md系统
- ✅ 掌握上下文层级
- ✅ 了解自动构建机制

### 专家级要点
- ✅ 深入上下文构建算法
- ✅ 掌握智能文件过滤
- ✅ 理解增量更新机制
- ✅ 掌握代码语义分析

---

**下一步：** 学习 [10 - Git集成](./10-git-integration.md) 🚀
