# 04 - 代理系统（Agents）

## 📋 模块介绍

代理系统是 Claude Code 的核心AI能力扩展机制，通过专业化代理完成特定任务。本章将深入讲解代理的设计、协作和开发方法。

---

## 🟢 入门级：代理基础认知

### 什么是代理？

代理（Agent）是**专业化AI助手**，每个代理都有明确的职责和能力范围。

**类比理解**：
- 👥 像团队中的专家（前端、后端、测试）
- 🎯 像工厂的专门工种（设计、生产、质检）
- 🏥 像医院的专科医生（外科、内科、眼科）

### 为什么需要代理？

```bash
# 通用Claude（什么都能做）
claude> 帮我审查代码、写测试、部署应用

问题：可能不够专业，遗漏细节

# 使用多个代理
claude> /code-review    # 代码审查代理
claude> /test-writer    # 测试编写代理
claude> /deploy         # 部署代理

优势：每个代理都是专家，质量更高
```

**代理的优势**：
- 🎯 **专业化** - 每个代理专注一个领域
- 🧠 **深度理解** - 针对性训练和配置
- 🔁 **可复用** - 多次使用保持一致性
- 🤝 **可协作** - 多个代理协同工作

### 官方代理示例

| 代理 | 职责 | 能力 |
|------|------|------|
| **code-reviewer** | 代码审查 | 检查质量、安全、最佳实践 |
| **test-engineer** | 测试工程师 | 编写测试、提高覆盖率 |
| **security-auditor** | 安全审计 | 发现漏洞、风险评估 |
| **documentation-writer** | 文档编写 | 生成文档、API参考 |
| **api-designer** | API设计 | 设计RESTful API、接口规范 |

### 代理 vs 命令 vs 插件

| 特性 | 命令 | 代理 | 插件 |
|------|------|------|------|
| **触发方式** | 手动输入 | 自动委派 | 安装启用 |
| **职责** | 执行特定操作 | 完成复杂任务 | 提供功能包 |
| **智能程度** | 低（预设流程） | 高（AI决策） | 中 |
| **复用性** | 高 | 高 | 极高 |
| **示例** | `/commit` | code-reviewer | code-review插件 |

### 如何使用代理？

#### 1. 自动委派

```bash
$ claude
claude> 审查这个文件的代码质量

# Claude自动选择code-reviewer代理
🔍 Code Reviewer 代理已激活
正在分析代码...
```

#### 2. 手动指定

```bash
claude> @code-reviewer 审查src/main.py

# 明确指定使用code-reviewer代理
```

#### 3. 多代理协作

```bash
claude> 完成这个功能开发

# Claude自动协调多个代理
1. @code-architect - 设计架构
2. @implementation-agent - 实现代码
3. @test-engineer - 编写测试
4. @code-reviewer - 审查代码
```

---

## 🟡 中级：代理开发与协作

### 代理定义格式

代理使用结构化Markdown定义：

```markdown
---
id: "code-reviewer"
name: "Code Reviewer"
role: "Quality Assurance"
description: "Specialized agent for code review"
version: "1.0.0"
permissions:
  - "file:read"
  - "git:read"
  - "git:diff"
capabilities:
  - "code-quality-check"
  - "security-audit"
  - "best-practice-validation"
tools:
  - "file:read"
  - "git:diff"
  - "git:log"
---

你是一个专业的代码审查专家。

## 职责
- 审查代码质量和安全性
- 遵循最佳实践
- 提供建设性反馈

## 审查标准

### 代码质量
- 代码可读性
- 性能优化
- 错误处理
- 命名规范

### 安全性
- SQL注入
- XSS漏洞
- 权限验证
- 敏感数据

### 最佳实践
- 设计模式
- 代码复用
- 注释文档
- 测试覆盖

## 工作流程

1. **分析变更**
   - 查看git diff
   - 理解代码意图
   - 识别影响范围

2. **检查问题**
   - 静态分析
   - 安全扫描
   - 性能评估

3. **生成报告**
   - 列出问题
   - 提供建议
   - 优先级排序

4. **验证修复**
   - 检查修复方案
   - 确认解决
   - 更新报告

## 输出格式

```markdown
## 代码审查报告

### 📊 概览
- 文件：{{file}}
- 变更行数：{{linesChanged}}
- 发现问题：{{issueCount}}个

### 🔍 发现的问题

#### 🔴 严重问题
{{#each criticalIssues}}
1. **{{title}}**
   - 位置：`{{location}}`
   - 描述：{{description}}
   - 修复：{{suggestion}}
{{/each}}

#### 🟡 一般问题
{{#each normalIssues}}
1. **{{title}}**
   - 位置：`{{location}}`
   - 描述：{{description}}
   - 修复：{{suggestion}}
{{/each}}

### ✅ 做得好的地方
- {{goodPoint1}}
- {{goodPoint2}}

### 📈 综合评分
- 代码质量：{{qualityScore}}/10
- 安全性：{{securityScore}}/10
- 可维护性：{{maintainabilityScore}}/10
```

## 注意事项
- 保持客观和专业
- 提供具体的改进建议
- 鼓励好的实践
- 解释问题的原因
```

### 代理Frontmatter字段详解

| 字段 | 类型 | 必需 | 说明 |
|------|------|------|------|
| `id` | string | ✅ | 代理唯一标识 |
| `name` | string | ✅ | 代理显示名称 |
| `role` | string | ✅ | 代理角色 |
| `description` | string | ✅ | 代理描述 |
| `permissions` | string[] | ✅ | 需要的权限 |
| `capabilities` | string[] | ✅ | 能力列表 |
| `tools` | string[] | ✅ | 可用工具 |
| `version` | string | ❌ | 版本号 |
| `model` | string | ❌ | 使用的AI模型 |

### 创建自定义代理

#### 步骤1：创建代理文件

```bash
mkdir -p .claude/agents
```

#### 步骤2：编写代理

创建 `.claude/agents/python-expert.md`：

```markdown
---
id: "python-expert"
name: "Python Expert"
role: "Python Development Specialist"
description: "Expert in Python development, best practices, and optimization"
permissions:
  - "file:read"
  - "file:write"
  - "bash:run"
capabilities:
  - "python-code-review"
  - "performance-optimization"
  - "code-refactoring"
tools:
  - "file:read"
  - "file:write"
  - "bash:run"
---

你是Python开发专家。

## 专长
- Python 3.8+ 最佳实践
- PEP 8 代码规范
- 性能优化
- 异步编程
- 类型注解

## 审查要点

### 代码风格
- 遵循PEP 8
- 使用类型注解
- 适当的注释
- 命名约定

### 性能
- 避免不必要的循环
- 使用生成器
- 合理使用缓存
- 异步操作

### 安全
- 输入验证
- SQL注入防护
- 敏感数据处理
- 依赖管理

## 帮助用户
1. 识别Python特定问题
2. 提供最佳实践建议
3. 优化性能
4. 修复常见错误
```

#### 步骤3：测试代理

```bash
$ claude
claude> 审查这个Python文件

# Python Expert 代理自动激活
🐍 Python Expert 已激活

正在分析 main.py...

发现以下问题：

1. ❌ 未使用类型注解
   位置：main.py:10
   建议：添加参数类型和返回值类型

2. ⚠️ 使用了不必要的列表推导
   位置：main.py:25
   建议：使用生成器表达式

3. ✅ 异常处理很好
   位置：main.py:40
```

### 代理协作机制

#### 1. 串行协作

```markdown
# 场景：功能开发
任务流：设计 → 实现 → 测试 → 审查

代理链：
1. @code-architect
   - 设计架构
   - 定义接口

2. @implementation-agent
   - 实现代码
   - 编写文档

3. @test-engineer
   - 编写测试
   - 提高覆盖率

4. @code-reviewer
   - 审查代码
   - 提出改进
```

#### 2. 并行协作

```markdown
# 场景：代码审查
多个代理同时审查不同方面

代理组：
1. @code-reviewer
   - 代码质量

2. @security-auditor
   - 安全漏洞

3. @performance-analyst
   - 性能问题

4. @documentation-checker
   - 文档完整性

并行执行 → 汇总结果 → 生成报告
```

#### 3. 主从协作

```markdown
# 场景：复杂任务
主代理协调，从代理执行

@orchestrator (主代理)
  ↓
  ├─ @frontend-developer (开发前端)
  ├─ @backend-developer (开发后端)
  └─ @test-engineer (编写测试)

主代理负责：
- 任务分解
- 协调进度
- 汇总结果
- 质量把控
```

---

## 🔴 专家级：代理系统深度剖析

### 代理注册系统

```typescript
class AgentRegistry {
  private agents: Map<string, Agent>;
  private capabilities: Map<string, Agent[]>;

  register(agent: Agent): void {
    // 注册代理
    this.agents.set(agent.id, agent);

    // 索引能力
    for (const capability of agent.capabilities) {
      if (!this.capabilities.has(capability)) {
        this.capabilities.set(capability, []);
      }
      this.capabilities.get(capability)!.push(agent);
    }
  }

  findAgent(id: string): Agent | null {
    return this.agents.get(id) || null;
  }

  findByCapability(capability: string): Agent[] {
    return this.capabilities.get(capability) || [];
  }

  findBestAgent(task: Task): Agent | null {
    // 1. 分析任务需求
    const requiredCapabilities = this.analyzeCapabilities(task);

    // 2. 查找匹配的代理
    const candidates: Agent[] = [];
    for (const capability of requiredCapabilities) {
      const agents = this.findByCapability(capability);
      candidates.push(...agents);
    }

    // 3. 评分排序
    const scored = candidates.map(agent => ({
      agent,
      score: this.score(agent, task)
    }));

    scored.sort((a, b) => b.score - a.score);

    // 4. 返回最佳代理
    return scored[0]?.agent || null;
  }

  private score(agent: Agent, task: Task): number {
    let score = 0;

    // 能力匹配度
    const matched = agent.capabilities.filter(c =>
      task.requiredCapabilities.includes(c)
    ).length;
    score += (matched / agent.capabilities.length) * 40;

    // 历史成功率
    score += agent.stats.successRate * 30;

    // 任务复杂度匹配
    if (agent.maxComplexity >= task.complexity) {
      score += 20;
    }

    // 优先级
    score += agent.priority * 10;

    return score;
  }
}
```

### 代理调度器

```typescript
class AgentScheduler {
  private queue: TaskQueue;
  private workers: Map<string, AgentWorker>;

  async schedule(task: Task): Promise<void> {
    // 1. 查找最佳代理
    const agent = this.registry.findBestAgent(task);
    if (!agent) {
      throw new Error('No suitable agent found');
    }

    // 2. 创建worker
    const worker = await this.createWorker(agent);

    // 3. 加入队列
    await this.queue.enqueue(task, agent);

    // 4. 分配worker
    await this.assignWorker(task, worker);
  }

  private async createWorker(agent: Agent): Promise<AgentWorker> {
    // 1. 初始化上下文
    const context = await this.initializeContext(agent);

    // 2. 创建worker
    const worker = new AgentWorker({
      agent,
      context,
      tools: this.getTools(agent)
    });

    // 3. 启动worker
    await worker.start();

    return worker;
  }

  private async initializeContext(agent: Agent): Promise<AgentContext> {
    return {
      id: generateId(),
      agentId: agent.id,
      state: {},
      history: [],
      permissions: agent.permissions,
      tools: agent.tools
    };
  }
}
```

### 代理通信

```typescript
class AgentMessenger {
  private channels: Map<string, MessageChannel>;

  async send(
    from: string,
    to: string,
    message: AgentMessage
  ): Promise<void> {
    const channel = this.channels.get(to);

    if (!channel) {
      throw new Error('Agent not found: ' + to);
    }

    // 验证权限
    if (!this.canSend(from, to)) {
      throw new Error('Permission denied');
    }

    // 发送消息
    await channel.send(message);

    // 记录日志
    this.log({
      from,
      to,
      message,
      timestamp: Date.now()
    });
  }

  async broadcast(
    from: string,
    message: AgentMessage
  ): Promise<void> {
    const agents = this.registry.getAllAgents();

    const promises = agents
      .filter(agent => agent.id !== from)
      .map(agent => this.send(from, agent.id, message));

    await Promise.all(promises);
  }

  subscribe(
    agentId: string,
    handler: MessageHandler
  ): void {
    const channel = this.channels.get(agentId);
    if (channel) {
      channel.subscribe(handler);
    }
  }
}
```

### 代理状态管理

```typescript
class AgentStateManager {
  private states: Map<string, AgentState>;

  async getState(agentId: string): Promise<AgentState> {
    return this.states.get(agentId) || this.createInitialState(agentId);
  }

  async updateState(
    agentId: string,
    update: Partial<AgentState>
  ): Promise<void> {
    const state = await this.getState(agentId);
    const updated = { ...state, ...update };

    this.states.set(agentId, updated);

    // 持久化
    await this.persist(agentId, updated);
  }

  async resetState(agentId: string): Promise<void> {
    const state = this.createInitialState(agentId);
    this.states.set(agentId, state);
    await this.persist(agentId, state);
  }

  private createInitialState(agentId: string): AgentState {
    return {
      agentId,
      status: 'idle',
      tasks: [],
      stats: {
        tasksCompleted: 0,
        tasksFailed: 0,
        avgDuration: 0,
        successRate: 1.0
      },
      lastActivity: Date.now()
    };
  }

  private async persist(
    agentId: string,
    state: AgentState
  ): Promise<void> {
    // 持久化到数据库或文件
    const path = path.join(
      this.stateDir,
      `${agentId}.json`
    );
    await fs.writeFile(path, JSON.stringify(state));
  }
}
```

### 代理性能监控

```typescript
class AgentMonitor {
  private metrics: Map<string, AgentMetrics>;

  async recordExecution(
    agentId: string,
    task: Task,
    result: ExecutionResult
  ): Promise<void> {
    const metrics = await this.getMetrics(agentId);

    // 记录执行
    metrics.executions.push({
      taskId: task.id,
      duration: result.duration,
      success: result.success,
      timestamp: Date.now()
    });

    // 更新统计
    metrics.totalExecutions++;
    if (result.success) {
      metrics.successfulExecutions++;
    } else {
      metrics.failedExecutions++;
    }

    // 计算成功率
    metrics.successRate =
      metrics.successfulExecutions / metrics.totalExecutions;

    // 计算平均时长
    const durations = metrics.executions
      .filter(e => e.success)
      .map(e => e.duration);
    metrics.avgDuration =
      durations.reduce((a, b) => a + b, 0) / durations.length;

    await this.saveMetrics(agentId, metrics);
  }

  async getPerformanceReport(
    agentId: string
  ): Promise<PerformanceReport> {
    const metrics = await this.getMetrics(agentId);

    return {
      agentId,
      totalExecutions: metrics.totalExecutions,
      successRate: metrics.successRate,
      avgDuration: metrics.avgDuration,
      topTasks: this.getTopTasks(metrics),
      commonErrors: this.getCommonErrors(metrics),
      recommendations: this.generateRecommendations(metrics)
    };
  }

  private generateRecommendations(
    metrics: AgentMetrics
  ): string[] {
    const recommendations: string[] = [];

    if (metrics.successRate < 0.8) {
      recommendations.push('成功率较低，建议检查错误处理逻辑');
    }

    if (metrics.avgDuration > 30000) {
      recommendations.push('平均执行时间较长，建议优化性能');
    }

    const errors = this.getCommonErrors(metrics);
    if (errors.length > 0) {
      recommendations.push(
        `常见错误：${errors[0].error} (出现${errors[0].count}次)`
      );
    }

    return recommendations;
  }
}
```

### 代理安全机制

```typescript
class AgentSecurity {
  private permissions: Map<string, Permission[]>;

  async checkPermission(
    agentId: string,
    action: string,
    resource: string
  ): Promise<boolean> {
    const permissions = this.permissions.get(agentId) || [];

    for (const permission of permissions) {
      if (permission.action === action) {
        return this.checkResourcePermission(
          permission,
          resource
        );
      }
    }

    return false;
  }

  private checkResourcePermission(
    permission: Permission,
    resource: string
  ): boolean {
    // 检查资源策略
    for (const policy of permission.resourcePolicies) {
      if (this.matchResource(policy.pattern, resource)) {
        return policy.allow;
      }
    }

    return permission.defaultAllow;
  }

  async auditAccess(
    agentId: string,
    action: string,
    resource: string
  ): Promise<void> {
    // 记录访问审计
    const auditLog = {
      agentId,
      action,
      resource,
      timestamp: Date.now(),
      result: await this.checkPermission(agentId, action, resource)
    };

    await this.logAudit(auditLog);
  }

  async detectAnomalies(agentId: string): Promise<Anomaly[]> {
    const logs = await this.getAuditLogs(agentId);
    const anomalies: Anomaly[] = [];

    // 检测异常访问频率
    const recentLogs = logs.filter(log =>
      Date.now() - log.timestamp < 60000
    );
    if (recentLogs.length > 1000) {
      anomalies.push({
        type: 'high_frequency',
        severity: 'warning',
        message: '访问频率异常高'
      });
    }

    // 检测权限提升尝试
    const deniedLogs = logs.filter(log => !log.result);
    if (deniedLogs.length > 10) {
      anomalies.push({
        type: 'permission_denied',
        severity: 'critical',
        message: '频繁的权限拒绝'
      });
    }

    return anomalies;
  }
}
```

---

## 📚 实战案例：开发多代理协作系统

### 需求：代码发布流程

```markdown
场景：发布新版本到生产环境

需要多个代理协作：
1. @code-reviewer - 审查代码
2. @test-runner - 运行测试
3. @security-scanner - 安全扫描
4. @documentation-updater - 更新文档
5. @deployment-agent - 部署到生产
```

### 实现

#### 1. 协调代理

```markdown
---
id: "release-coordinator"
name: "Release Coordinator"
role: "Release Orchestration"
description: "Coordinate release process with multiple agents"
permissions:
  - "agent:invoke"
capabilities:
  - "release-orchestration"
  - "task-coordination"
---

你是发布协调员。

## 职责
- 协调发布流程
- 管理代理协作
- 监控发布进度
- 处理异常情况

## 发布流程

### 阶段1：准备
1. 调用 @code-reviewer 审查代码
2. 调用 @test-runner 运行测试
3. 调用 @security-scanner 安全扫描

### 阶段2：验证
- 确认所有检查通过
- 准备发布说明
- 更新文档

### 阶段3：部署
1. 调用 @deployment-agent 部署
2. 监控部署状态
3. 验证部署结果

### 阶段4：发布后
- 更新版本信息
- 通知团队
- 监控线上状态
```

#### 2. 使用流程

```bash
$ claude
claude> 发布 v1.0.0 到生产环境

🚀 Release Coordinator 已激活

### 阶段1：代码审查
📋 调用 code-reviewer...
✅ 代码审查通过

### 阶段2：测试
🧪 调用 test-runner...
✅ 所有测试通过

### 阶段3：安全扫描
🔒 调用 security-scanner...
✅ 未发现安全漏洞

### 阶段4：更新文档
📝 调用 documentation-updater...
✅ 文档已更新

### 阶段5：部署
🚀 调用 deployment-agent...
✅ 部署成功

### 发布完成！
版本：v1.0.0
时间：2026-04-14 12:00:00
状态：✅ 成功
```

---

## ✅ 章节总结

### 入门级要点
- ✅ 理解代理的概念
- ✅ 掌握基本使用方法
- ✅ 了解官方代理示例

### 中级要点
- ✅ 掌握代理定义格式
- ✅ 理解代理协作机制
- ✅ 学会创建自定义代理
- ✅ 掌握主从、串行、并行协作

### 专家级要点
- ✅ 深入代理注册系统
- ✅ 掌握调度和通信机制
- ✅ 理解状态管理和监控
- ✅ 掌握安全机制

---

**下一步：** 学习 [05 - 技能系统](./05-skill-system.md) 🚀
