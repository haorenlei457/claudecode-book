# 05 - 技能系统（Skills）

## 📋 模块介绍

技能系统是 Claude Code 的可复用能力包，类似于"专业技能"或"工具包"。本章将深入讲解技能的设计、开发和使用方法。

---

## 🟢 入门级：技能基础认知

### 什么是技能？

技能（Skill）是**封装的专业能力**，可以在需要时自动调用。

**类比理解**：
- 🛠️ 像工具箱中的专业工具（螺丝刀、扳手、电钻）
- 🎓 像专业技能（Python、JavaScript、机器学习）
- 📦 像软件包（npm包、pip库、gem）

### 技能 vs 插件 vs 代理

| 特性 | 技能 | 插件 | 代理 |
|------|------|------|------|
| **颗粒度** | 小（单一能力） | 大（完整功能） | 中（专家角色） |
| **触发方式** | 自动 | 手动/自动 | 手动/委派 |
| **智能程度** | 低（预设规则） | 高（完整逻辑） | 极高（AI决策） |
| **复用性** | 极高 | 高 | 高 |
| **示例** | 日志分析 | 代码审查插件 | 代码审查代理 |

### 官方技能示例

| 技能 | 功能 | 触发条件 |
|------|------|----------|
| **code-review** | 代码审查 | 提到"审查"、"检查质量" |
| **test-generation** | 生成测试 | 提到"测试"、"覆盖率" |
| **documentation** | 生成文档 | 提到"文档"、"注释" |
| **refactoring** | 代码重构 | 提到"重构"、"优化" |
| **debugging** | 调试辅助 | 提到"调试"、"错误" |

### 如何使用技能？

#### 1. 自动触发

```bash
$ claude
claude> 帮我审查这段代码的质量

# Claude自动检测需求，加载code-review技能
🔍 加载技能：code-review
正在审查代码...
```

#### 2. 手动指定

```bash
claude> 使用 test-generation 技能为这个函数编写测试

# 明确指定技能
✓ 已加载技能：test-generation
生成测试中...
```

#### 3. 查看可用技能

```bash
claude> /skills list

可用技能：
  - code-review (代码审查)
  - test-generation (测试生成)
  - documentation (文档生成)
  - refactoring (代码重构)
  - debugging (调试辅助)
```

---

## 🟡 中级：技能开发与设计

### 技能定义格式

技能使用 Markdown + YAML frontmatter 定义：

```markdown
---
name: "code-review"
description: "Automated code review capabilities"
version: "1.0.0"
category: "quality"
triggers:
  - "review code"
  - "check quality"
  - "audit security"
  "检查代码"
  - "审查质量"
permissions:
  - "file:read"
  - "git:diff"
author: "Anthropic"
---

# 代码审查技能

## 使用场景
当用户请求代码审查时自动触发

## 审查流程

### 1. 静态分析
```python
# 识别常见问题模式
issues = [
    'unused-variables',
    'missing-error-handling',
    'sql-injection-risk',
    'xss-vulnerability',
    'hardcoded-secrets'
]
```

### 2. 最佳实践检查
- 遵循项目编码规范
- 验证命名约定
- 检查代码复杂度
- 验证注释完整性

### 3. 性能优化建议
- 识别N+1查询
- 建议缓存策略
- 优化算法复杂度
- 减少内存使用

### 4. 安全扫描
- 依赖漏洞检查
- 敏感数据泄露
- 权限验证
- 输入验证

## 输出格式

```markdown
## 代码审查报告

### 🔍 发现的问题

#### 🔴 严重问题
{{#each criticalIssues}}
1. **{{type}}**
   - 位置：`{{location}}`
   - 描述：{{description}}
   - 修复：{{suggestion}}
   - 严重性：{{severity}}
{{/each}}

#### 🟡 一般问题
{{#each normalIssues}}
1. **{{type}}**
   - 位置：`{{location}}`
   - 描述：{{description}}
   - 修复：{{suggestion}}
   - 严重性：{{severity}}
{{/each}}

### ✅ 做得好的地方
- {{goodPoint1}}
- {{goodPoint2}}
- {{goodPoint3}}

### 📊 综合评分
- 代码质量：{{qualityScore}}/10
- 安全性：{{securityScore}}/10
- 可维护性：{{maintainabilityScore}}/10
- 性能：{{performanceScore}}/10

### 💡 改进建议
{{recommendations}}
```

## 注意事项
- 保持客观和专业
- 提供具体的改进建议
- 鼓励好的实践
- 解释问题的原因
```

### 技能目录结构

```
my-skill/
├── SKILL.md              # 技能定义（必需）
├── examples.md           # 使用示例（推荐）
├── templates/            # 模板文件（可选）
│   ├── report.md
│   └── checklist.md
└── scripts/              # 辅助脚本（可选）
    └── analyzer.py
```

### 创建自定义技能

#### 步骤1：创建技能目录

```bash
mkdir -p .claude/skills/log-analyzer/templates
```

#### 步骤2：编写技能定义

创建 `.claude/skills/log-analyzer/SKILL.md`：

```markdown
---
name: "log-analyzer"
description: "Analyze application logs and identify issues"
version: "1.0.0"
category: "analysis"
triggers:
  - "analyze logs"
  - "check errors"
  - "review logs"
  - "分析日志"
  - "检查错误"
permissions:
  - "file:read"
author: "Your Name"
---

# 日志分析技能

## 使用场景
当用户需要分析应用日志时自动触发

## 分析能力

### 1. 错误识别
- 识别错误级别（ERROR, FATAL）
- 统计错误频率
- 识别错误模式

### 2. 性能分析
- 分析响应时间
- 识别慢查询
- 检测性能瓶颈

### 3. 趋势分析
- 错误趋势
- 流量模式
- 资源使用

### 4. 异常检测
- 突发错误
- 异常流量
- 资源异常

## 分析流程

1. **读取日志**
   - 支持多种格式（JSON, Common Log, 自定义）
   - 解析时间戳和级别
   - 提取关键字段

2. **模式识别**
   - 错误模式匹配
   - 异常行为检测
   - 关联分析

3. **统计汇总**
   - 错误统计
   - 性能指标
   - 趋势数据

4. **生成报告**
   - 问题列表
   - 建议措施
   - 优先级排序
```

#### 步骤3：创建模板

创建 `.claude/skills/log-analyzer/templates/report.md`：

```markdown
# 日志分析报告

## 📊 概览
- 分析时间：{{analysisTime}}
- 日志文件：{{logFile}}
- 总行数：{{totalLines}}
- 时间范围：{{timeRange}}

## 🔴 错误汇总

| 错误类型 | 次数 | 占比 | 首次出现 |
|----------|------|------|----------|
{{#each errorStats}}
| {{type}} | {{count}} | {{percentage}}% | {{firstSeen}} |
{{/each}}

## 📈 性能指标

- 平均响应时间：{{avgResponseTime}}ms
- P95 响应时间：{{p95ResponseTime}}ms
- P99 响应时间：{{p99ResponseTime}}ms
- QPS：{{qps}}

## ⚠️ 关键问题

{{#each criticalIssues}}
### {{title}}
- 严重性：{{severity}}
- 首次出现：{{firstSeen}}
- 出现次数：{{occurrences}}
- 描述：{{description}}
- 建议：{{recommendation}}
{{/each}}

## 💡 优化建议

{{#each recommendations}}
1. {{title}}
   - 优先级：{{priority}}
   - 预期效果：{{expectedImpact}}
{{/each}}
```

#### 步骤4：测试技能

```bash
$ claude
claude> 分析 logs/app.log 文件的错误

✓ 加载技能：log-analyzer

# 日志分析报告

## 📊 概览
- 分析时间：2026-04-14 12:00:00
- 日志文件：logs/app.log
- 总行数：10,000
- 时间范围：2026-04-13 12:00 ~ 2026-04-14 12:00

## 🔴 错误汇总

| 错误类型 | 次数 | 占比 | 首次出现 |
|----------|------|------|----------|
| DatabaseConnectionError | 45 | 2.3% | 2026-04-14 08:00 |
| TimeoutError | 23 | 1.2% | 2026-04-14 10:00 |

## ⚠️ 关键问题

### 数据库连接池耗尽
- 严重性：高
- 首次出现：2026-04-14 08:00
- 出现次数：45
- 描述：数据库连接池已满，无法获取新连接
- 建议：增加连接池大小或优化查询

## 💡 优化建议

1. 优化数据库查询
   - 优先级：高
   - 预期效果：减少30%的数据库连接
```

---

## 🔴 专家级：技能系统深度剖析

### 技能加载器

```typescript
class SkillLoader {
  private skills: Map<string, Skill>;

  async load(skillPath: string): Promise<Skill> {
    // 1. 读取 SKILL.md
    const manifestPath = path.join(skillPath, 'SKILL.md');
    const content = await fs.readFile(manifestPath, 'utf-8');

    // 2. 解析 frontmatter
    const { data: frontmatter, content: body } =
      matter(content);

    // 3. 验证元数据
    this.validateManifest(frontmatter);

    // 4. 加载模板
    const templates = await this.loadTemplates(skillPath);

    // 5. 加载脚本
    const scripts = await this.loadScripts(skillPath);

    // 6. 创建技能对象
    const skill: Skill = {
      ...frontmatter,
      content: body,
      templates,
      scripts,
      path: skillPath
    };

    return skill;
  }

  private async loadTemplates(
    skillPath: string
  ): Promise<Map<string, string>> {
    const templates = new Map<string, string>();
    const templateDir = path.join(skillPath, 'templates');

    if (await fs.pathExists(templateDir)) {
      const files = await fs.readdir(templateDir);
      for (const file of files) {
        const content = await fs.readFile(
          path.join(templateDir, file),
          'utf-8'
        );
        const name = path.basename(file, path.extname(file));
        templates.set(name, content);
      }
    }

    return templates;
  }
}
```

### 技能匹配器

```typescript
class SkillMatcher {
  private skills: Skill[];

  async match(input: string): Promise<Skill[]> {
    const matches: Skill[] = [];

    for (const skill of this.skills) {
      const score = this.calculateScore(skill, input);

      // 阈值：0.6以上认为匹配
      if (score > 0.6) {
        matches.push({ skill, score });
      }
    }

    // 按分数排序
    matches.sort((a, b) => b.score - a.score);

    return matches.map(m => m.skill);
  }

  private calculateScore(skill: Skill, input: string): number {
    const normalizedInput = input.toLowerCase();
    let score = 0;

    // 1. 触发词匹配
    for (const trigger of skill.triggers) {
      if (normalizedInput.includes(trigger.toLowerCase())) {
        score += 0.5;
      }
    }

    // 2. 描述匹配
    const descWords = skill.description.toLowerCase().split(/\s+/);
    const inputWords = normalizedInput.split(/\s+/);
    const commonWords = descWords.filter(w => inputWords.includes(w));
    score += (commonWords.length / descWords.length) * 0.3;

    // 3. 分类匹配
    if (normalizedInput.includes(skill.category)) {
      score += 0.2;
    }

    return Math.min(score, 1.0);
  }
}
```

### 技能执行器

```typescript
class SkillExecutor {
  async execute(skill: Skill, context: Context): Promise<void> {
    // 1. 准备上下文
    const executionContext = await this.prepareContext(skill, context);

    // 2. 执行预处理脚本
    await this.executePreScripts(skill, executionContext);

    // 3. 执行技能逻辑
    const result = await this.executeSkillLogic(skill, executionContext);

    // 4. 渲染输出
    const output = await this.renderOutput(skill, result);

    // 5. 执行后处理脚本
    await this.executePostScripts(skill, executionContext, result);

    return output;
  }

  private async prepareContext(
    skill: Skill,
    context: Context
  ): Promise<ExecutionContext> {
    return {
      ...context,
      skill: {
        name: skill.name,
        version: skill.version,
        path: skill.path
      },
      templates: skill.templates,
      scripts: skill.scripts
    };
  }

  private async executeSkillLogic(
    skill: Skill,
    context: ExecutionContext
  ): Promise<any> {
    // 解析技能内容，提取步骤
    const steps = this.parseSteps(skill.content);

    // 执行每个步骤
    const results = [];
    for (const step of steps) {
      const result = await this.executeStep(step, context);
      results.push(result);
    }

    return results;
  }
}
```

### 技能缓存

```typescript
class SkillCache {
  private cache: LRUCache<string, CachedSkill>;

  async get(skillId: string): Promise<Skill | null> {
    const cached = this.cache.get(skillId);

    if (!cached) return null;

    // 检查是否过期
    if (Date.now() - cached.timestamp > cached.ttl) {
      this.cache.delete(skillId);
      return null;
    }

    return cached.skill;
  }

  async set(skill: Skill, ttl: number = 3600000): Promise<void> {
    this.cache.set(skill.name, {
      skill,
      timestamp: Date.now(),
      ttl
    });
  }

  invalidate(skillId: string): void {
    this.cache.delete(skillId);
  }

  clear(): void {
    this.cache.clear();
  }
}
```

---

## 📚 实战案例：开发API文档生成技能

### 需求
- 📝 从代码生成API文档
- 🎯 支持多种框架（Express, FastAPI, Django）
- 📊 生成OpenAPI规范
- 🎨 支持Markdown和HTML输出

### 实现

#### 1. 技能定义

```markdown
---
name: "api-documentation"
description: "Generate API documentation from code"
version: "1.0.0"
category: "documentation"
triggers:
  - "generate api docs"
  - "create documentation"
  - "api reference"
  - "生成API文档"
permissions:
  - "file:read"
  - "file:write"
author: "Your Name"
---

# API文档生成技能

## 使用场景
当用户需要生成API文档时自动触发

## 支持的框架

### JavaScript/TypeScript
- Express.js
- Koa.js
- Fastify
- NestJS

### Python
- FastAPI
- Flask
- Django REST Framework

### 其他
- Go Gin
- Java Spring Boot

## 生成流程

1. **代码分析**
   - 识别路由定义
   - 解析参数和返回值
   - 提取JSDoc/注释

2. **文档生成**
   - 生成Markdown文档
   - 生成OpenAPI规范
   - 生成HTML页面

3. **输出格式**
   - Markdown
   - HTML
   - JSON (OpenAPI)
```

#### 2. 模板

```markdown
# {{projectName}} API Documentation

## 概述
{{description}}

## 认证
{{#if authentication}}
- 类型：{{authentication.type}}
- 说明：{{authentication.description}}
{{/if}}

## 端点列表

{{#each endpoints}}
### {{method}} {{path}}

**说明**
{{description}}

**请求参数**

| 参数 | 类型 | 必需 | 说明 |
|------|------|------|------|
{{#each parameters}}
| {{name}} | {{type}} | {{required}} | {{description}} |
{{/each}}

**响应示例**

```json
{{{responseExample}}}
```

**错误响应**

| 状态码 | 说明 |
|--------|------|
{{#each errorResponses}}
| {{code}} | {{description}} |
{{/each}}

{{/each}}
```

#### 3. 使用示例

```bash
$ claude
claude> 为这个Express应用生成API文档

✓ 加载技能：api-documentation

分析代码中...
发现 15 个端点

生成文档...

✅ API文档已生成：
- README.md (Markdown版本)
- openapi.json (OpenAPI规范)
- docs/index.html (HTML版本)
```

---

## ✅ 章节总结

### 入门级要点
- ✅ 理解技能的概念
- ✅ 掌握基本使用方法
- ✅ 了解官方技能示例

### 中级要点
- ✅ 掌握技能定义格式
- ✅ 理解触发机制
- ✅ 学会创建自定义技能
- ✅ 掌握模板系统

### 专家级要点
- ✅ 深入加载和匹配机制
- ✅ 掌握执行引擎
- ✅ 理解缓存策略
- ✅ 掌握性能优化

---

**下一步：** 学习 [06 - 钩子系统](./06-hook-system.md) 🚀
