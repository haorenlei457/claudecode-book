# 02 - 插件系统

## 📋 模块介绍

插件系统是 Claude Code 的核心扩展机制，理解它对于自定义功能、开发插件和深度定制至关重要。本章将深入讲解插件的工作原理、开发方法和最佳实践。

---

## 🟢 入门级：插件基础认知

### 什么是插件？

插件（Plugin）是**扩展 Claude Code 功能的模块化组件**，就像给浏览器安装扩展一样。

**简单理解**：
- 🎯 **类比**：像 VS Code 的插件、Chrome 的扩展
- 📦 **组成**：命令 + 代理 + 技能 + 钩子
- 🔌 **作用**：为特定任务提供专业能力

### 插件能做什么？

```markdown
✅ 添加新命令（如 /commit, /review）
✅ 创建专业化 AI 代理
✅ 提供可复用技能
✅ 响应事件（自动执行）
✅ 连接外部工具和服务
```

### 官方插件示例

| 插件名 | 功能 | 用途 |
|--------|------|------|
| **code-review** | 代码审查 | 自动审查PR质量 |
| **commit-commands** | Git命令 | 简化提交流程 |
| **feature-dev** | 功能开发 | 7步开发流程 |
| **plugin-dev** | 插件开发 | 创建插件的工具 |
| **hookify** | Hook管理 | 轻松创建钩子 |

### 如何使用插件？

#### 方法1：安装官方插件

```bash
$ claude
claude> /plugin install code-review
✅ 已安装 code-review 插件

claude> /code-review
🔍 开始代码审查...
```

#### 方法2：手动安装

```bash
# 1. 下载插件
git clone https://github.com/anthropics/claude-code.git
cd claude-code/plugins/code-review

# 2. 复制到项目
cp -r .claude ~/.claude/
cp -r commands skills agents ~/.claude/

# 3. 配置 settings.json
cat > ~/.claude/settings.json << EOF
{
  "plugins": ["code-review"]
}
EOF
```

#### 方法3：项目级插件

```bash
# 在项目目录中创建插件目录
mkdir -p .claude/plugins/my-plugin

# 插件会自动加载
```

### 插件文件结构

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json          # 插件元数据（必需）
├── commands/                # 命令（可选）
│   └── my-command.md
├── agents/                  # 代理（可选）
│   └── my-agent.md
├── skills/                  # 技能（可选）
│   └── my-skill/
│       ├── SKILL.md
│       └── examples.md
├── hooks/                   # 钩子（可选）
│   └── my-hook.sh
├── .mcp.json                # MCP配置（可选）
└── README.md                # 说明文档（推荐）
```

---

## 🟡 中级：插件开发与工作原理

### 插件元数据详解

每个插件必须包含 `plugin.json`：

```json
{
  "name": "my-awesome-plugin",
  "version": "1.0.0",
  "description": "我的第一个插件",
  "author": "Your Name",
  "type": "plugin",
  "main": "./my-plugin.claude",
  "category": "development",
  "keywords": ["productivity", "automation"],
  "dependencies": {
    "anthropic": "^0.27.0"
  },
  "exports": {
    "commands": ["./commands/*.md"],
    "agents": ["./agents/*.md"],
    "skills": ["./skills/*"]
  },
  "permissions": [
    "file:read",
    "file:write",
    "git:read",
    "git:write"
  ],
  "settings": {
    "enabledByDefault": true,
    "autoUpdate": true
  },
  "engines": {
    "claude": ">=1.0.0"
  }
}
```

**字段说明**：

| 字段 | 必需 | 说明 |
|------|------|------|
| `name` | ✅ | 插件唯一标识 |
| `version` | ✅ | 语义化版本号 |
| `description` | ✅ | 简短描述 |
| `type` | ✅ | 必须是 "plugin" |
| `exports` | ✅ | 导出的组件 |
| `permissions` | ✅ | 需要的权限 |
| `dependencies` | ❌ | 依赖的包 |
| `settings` | ❌ | 默认配置 |

### 插件加载机制

```typescript
class PluginLoader {
  async loadPlugin(pluginPath: string): Promise<Plugin> {
    // 1. 读取 plugin.json
    const manifest = await this.readManifest(pluginPath);

    // 2. 验证插件
    this.validate(manifest);

    // 3. 加载导出的组件
    const components = await this.loadComponents(manifest);

    // 4. 检查权限
    await this.checkPermissions(manifest.permissions);

    // 5. 初始化插件
    const plugin: Plugin = {
      ...manifest,
      components,
      state: 'loaded'
    };

    return plugin;
  }

  private async loadComponents(manifest: PluginManifest): Promise<Components> {
    const components: Components = {
      commands: [],
      agents: [],
      skills: [],
      hooks: []
    };

    // 加载命令
    if (manifest.exports.commands) {
      for (const pattern of manifest.exports.commands) {
        const files = glob(pattern, { cwd: manifest.dir });
        for (const file of files) {
          components.commands.push(await this.parseCommand(file));
        }
      }
    }

    // 类似加载 agents, skills, hooks...

    return components;
  }

  private validate(manifest: PluginManifest): void {
    // 检查必需字段
    if (!manifest.name || !manifest.version) {
      throw new Error('Invalid plugin manifest');
    }

    // 检查版本兼容性
    const claudeVersion = getClaudeVersion();
    if (!semver.satisfies(claudeVersion, manifest.engines.claude)) {
      throw new Error('Plugin requires Claude >= ' + manifest.engines.claude);
    }
  }
}
```

### 插件生命周期

```
┌─────────────┐
│  Discovery  │ ← 发现插件（扫描目录）
└─────────────┘
      ↓
┌─────────────┐
│ Validation  │ ← 验证元数据和权限
└─────────────┘
      ↓
┌─────────────┐
│  Loading    │ ← 加载组件和资源
└─────────────┘
      ↓
┌─────────────┐
│ Activation  │ ← 激活插件（注册命令、代理）
└─────────────┘
      ↓
┌─────────────┐
│  Active     │ ← 插件运行中
└─────────────┘
      ↓
┌─────────────┐
│ Deactivation│ ← 停用插件
└─────────────┘
      ↓
┌─────────────┐
│ Unloading   │ ← 卸载插件
└─────────────┘
```

### 创建你的第一个插件

#### 步骤1：创建项目结构

```bash
mkdir -p my-first-plugin/.claude-plugin
mkdir -p my-first-plugin/commands
```

#### 步骤2：编写 plugin.json

```json
{
  "name": "hello-world",
  "version": "1.0.0",
  "description": "我的第一个插件 - 问候世界",
  "author": "Your Name",
  "type": "plugin",
  "exports": {
    "commands": ["./commands/*.md"]
  },
  "permissions": []
}
```

#### 步骤3：创建命令

创建 `commands/hello.md`：

```markdown
---
name: "hello"
description: "Say hello to the world"
---

你好！这是一个简单的命令。

让我们演示如何创建一个基本的命令：

## 基础使用
```bash
claude> /hello
```

## 参数传递
用户可以在命令后添加参数：
```bash
claude> /hello World
```

你可以通过 {{args}} 访问参数。

## 实际应用
这个插件展示了：
1. 如何定义命令
2. 如何编写命令描述
3. 如何处理用户输入

继续学习更多高级功能！
```

#### 步骤4：安装插件

```bash
# 方法1：复制到全局配置
cp -r my-first-plugin ~/.claude/plugins/

# 方法2：复制到项目
cp -r my-first-plugin .claude/plugins/

# 方法3：在 settings.json 中配置
cat > ~/.claude/settings.json << EOF
{
  "plugins": [
    "~/.claude/plugins/hello-world"
  ]
}
EOF
```

#### 步骤5：测试

```bash
$ claude
claude> /hello
你好！这是一个简单的命令...
```

### 插件依赖管理

```json
{
  "dependencies": {
    "anthropic": "^0.27.0",
    "axios": "^1.6.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "jest": "^29.0.0"
  }
}
```

**依赖解决策略**：

```typescript
class DependencyResolver {
  async resolve(dependencies: Record<string, string>): Promise<void> {
    for (const [name, version] of Object.entries(dependencies)) {
      // 1. 检查是否已安装
      const installed = await this.checkInstalled(name);

      if (!installed || !semver.satisfies(installed.version, version)) {
        // 2. 安装依赖
        await this.install(name, version);
      }

      // 3. 验证依赖
      await this.validate(name);
    }
  }

  private async install(name: string, version: string): Promise<void> {
    // 使用 npm/yarn/pnpm 安装
    const packageManager = detectPackageManager();
    await exec(`${packageManager} install ${name}@${version}`);
  }
}
```

---

## 🔴 专家级：插件系统深度剖析

### 插件系统核心架构

```typescript
interface PluginSystem {
  // 插件管理
  loadPlugin(path: string): Promise<Plugin>;
  unloadPlugin(pluginId: string): Promise<void>;
  reloadPlugin(pluginId: string): Promise<void>;
  getPlugin(pluginId: string): Plugin | null;

  // 组件注册
  registerCommand(command: Command): void;
  registerAgent(agent: Agent): void;
  registerSkill(skill: Skill): void;
  registerHook(hook: Hook): void;

  // 事件系统
  emit(event: string, data: any): void;
  on(event: string, handler: Function): void;

  // 权限管理
  checkPermission(pluginId: string, permission: string): boolean;
  grantPermission(pluginId: string, permission: string): void;
}
```

### 插件沙箱隔离

```typescript
class PluginSandbox {
  private context: SandboxContext;

  createSandbox(plugin: Plugin): SandboxContext {
    const context: SandboxContext = {
      // 受限的 API
      fs: this.createRestrictedFS(plugin),
      http: this.createRestrictedHTTP(plugin),
      child_process: this.createRestrictedProcess(plugin),

      // 插件专用存储
      storage: this.createPluginStorage(plugin.id),

      // 事件总线
      events: this.createEventBus(plugin),

      // 日志
      logger: this.createLogger(plugin.id)
    };

    return context;
  }

  private createRestrictedFS(plugin: Plugin): any {
    const allowedPaths = this.getAllowedPaths(plugin);

    return {
      readFile: (path: string) => {
        if (!this.isPathAllowed(path, allowedPaths)) {
          throw new Error('Access denied');
        }
        return fs.readFile(path);
      },
      writeFile: (path: string, data: any) => {
        if (!this.isPathAllowed(path, allowedPaths)) {
          throw new Error('Access denied');
        }
        return fs.writeFile(path, data);
      }
    };
  }
}
```

### 插件通信机制

#### 1. 插件间消息传递

```typescript
class PluginMessenger {
  private channels: Map<string, Channel>;

  async send(from: string, to: string, message: Message): Promise<void> {
    const channel = this.channels.get(to);

    if (!channel) {
      throw new Error('Plugin not found: ' + to);
    }

    // 检查权限
    if (!this.canSend(from, to)) {
      throw new Error('Permission denied');
    }

    // 发送消息
    await channel.deliver(message);
  }

  subscribe(pluginId: string, handler: MessageHandler): void {
    const channel = this.channels.getOrCreate(pluginId);
    channel.subscribe(handler);
  }
}
```

#### 2. 共享状态管理

```typescript
class SharedState {
  private state: Map<string, any>;
  private locks: Map<string, Lock>;

  async get(key: string, pluginId: string): Promise<any> {
    // 检查访问权限
    if (!this.hasAccess(pluginId, key)) {
      throw new Error('Access denied');
    }

    return this.state.get(key);
  }

  async set(key: string, value: any, pluginId: string): Promise<void> {
    // 获取锁
    const lock = await this.acquireLock(key);

    try {
      this.state.set(key, value);
    } finally {
      lock.release();
    }
  }

  private async acquireLock(key: string): Promise<Lock> {
    // 分布式锁实现
    const lock = new Lock(key);
    await lock.acquire();
    this.locks.set(key, lock);
    return lock;
  }
}
```

### 插件热更新

```typescript
class PluginHotReload {
  private watchers: Map<string, FSWatcher>;

  watch(plugin: Plugin): void {
    const watcher = chokidar.watch(plugin.dir, {
      ignored: /node_modules/
    });

    watcher.on('change', async (path) => {
      // 防抖
      await this.debounce(() => {
        this.reloadPlugin(plugin.id);
      }, 1000);
    });

    this.watchers.set(plugin.id, watcher);
  }

  private async reloadPlugin(pluginId: string): Promise<void> {
    const plugin = this.getPlugin(pluginId);

    // 1. 卸载旧组件
    await this.unloadComponents(plugin);

    // 2. 重新加载
    const updated = await this.loadPlugin(plugin.dir);

    // 3. 保留状态（如果可能）
    if (this.canPreserveState(plugin, updated)) {
      updated.state = plugin.state;
    }

    // 4. 通知其他插件
    this.emit('plugin:reloaded', { pluginId });
  }
}
```

### 插件性能分析

```typescript
class PluginProfiler {
  private metrics: Map<string, PluginMetrics>;

  async measure<T>(
    pluginId: string,
    operation: string,
    fn: () => Promise<T>
  ): Promise<T> {
    const start = performance.now();
    const memoryBefore = process.memoryUsage();

    try {
      return await fn();
    } finally {
      const end = performance.now();
      const memoryAfter = process.memoryUsage();

      // 记录指标
      const metrics: PluginMetrics = {
        duration: end - start,
        memoryDelta: memoryAfter.heapUsed - memoryBefore.heapUsed,
        timestamp: Date.now()
      };

      this.record(pluginId, operation, metrics);
    }
  }

  getProfile(pluginId: string): Profile {
    const metrics = this.metrics.get(pluginId);

    return {
      totalCalls: metrics.length,
      avgDuration: this.average(metrics, 'duration'),
      maxDuration: this.max(metrics, 'duration'),
      memoryUsage: this.sum(metrics, 'memoryDelta'),
      hotspots: this.identifyHotspots(metrics)
    };
  }
}
```

### 插件安全策略

#### 1. 代码签名验证

```typescript
class PluginValidator {
  async verify(plugin: Plugin): Promise<boolean> {
    // 1. 检查签名
    const signature = await this.readSignature(plugin);
    if (!signature) {
      console.warn('Plugin is not signed');
      return false;
    }

    // 2. 验证签名
    const isValid = await crypto.verify({
      data: this.getPluginHash(plugin),
      signature: signature,
      publicKey: await this.getPublicKey(plugin.author)
    });

    if (!isValid) {
      throw new Error('Invalid plugin signature');
    }

    // 3. 检查撤销列表
    if (await this.isRevoked(plugin.id)) {
      throw new Error('Plugin has been revoked');
    }

    return true;
  }

  private getPluginHash(plugin: Plugin): string {
    // 计算插件的哈希值
    const files = glob('**/*', { cwd: plugin.dir });
    const hashes = files.map(f => hashFile(f));
    return hash(hashes.join(''));
  }
}
```

#### 2. 运行时权限控制

```typescript
class PermissionMonitor {
  private auditLog: AuditLog[];

  async checkAndLog(
    pluginId: string,
    permission: string,
    resource: string
  ): Promise<boolean> {
    const granted = this.hasPermission(pluginId, permission);

    // 记录审计日志
    this.auditLog.push({
      pluginId,
      permission,
      resource,
      granted,
      timestamp: Date.now()
    });

    return granted;
  }

  analyzeSuspiciousActivity(pluginId: string): Report {
    const logs = this.auditLog.filter(l => l.pluginId === pluginId);

    return {
      totalRequests: logs.length,
      deniedRequests: logs.filter(l => !l.granted).length,
      suspiciousPatterns: this.detectPatterns(logs),
      recommendations: this.generateRecommendations(logs)
    };
  }

  private detectPatterns(logs: AuditLog[]): Pattern[] {
    const patterns: Pattern[] = [];

    // 检测异常频率
    const recent = logs.filter(l =>
      Date.now() - l.timestamp < 60000
    );
    if (recent.length > 100) {
      patterns.push({
        type: 'high_frequency',
        severity: 'warning',
        message: '插件请求频率异常高'
      });
    }

    // 检测权限提升尝试
    if (this.hasEscalationAttempts(logs)) {
      patterns.push({
        type: 'privilege_escalation',
        severity: 'critical',
        message: '检测到权限提升尝试'
      });
    }

    return patterns;
  }
}
```

### 插件市场集成

```typescript
class MarketplaceClient {
  async search(query: string): Promise<PluginInfo[]> {
    const response = await fetch(
      `https://marketplace.claude.com/api/plugins?q=${encodeURIComponent(query)}`
    );

    const results = await response.json();

    return results.map(r => ({
      id: r.id,
      name: r.name,
      version: r.version,
      description: r.description,
      author: r.author,
      downloads: r.downloads,
      rating: r.rating,
      tags: r.tags
    }));
  }

  async install(pluginId: string): Promise<void> {
    // 1. 获取插件信息
    const info = await this.getPluginInfo(pluginId);

    // 2. 验证插件
    await this.verify(info);

    // 3. 下载插件
    const archive = await this.download(info.downloadUrl);

    // 4. 解压
    await this.extract(archive, this.getPluginDir(info.id));

    // 5. 加载插件
    await this.loadPlugin(info.id);
  }

  async update(pluginId: string): Promise<void> {
    // 1. 检查更新
    const current = this.getInstalledPlugin(pluginId);
    const latest = await this.getLatestVersion(pluginId);

    if (semver.gte(current.version, latest.version)) {
      console.log('Already up to date');
      return;
    }

    // 2. 备份当前版本
    await this.backup(pluginId);

    // 3. 安装新版本
    await this.install(pluginId);

    // 4. 迁移配置
    await this.migrateConfig(pluginId, current.version, latest.version);
  }
}
```

### 插件测试框架

```typescript
class PluginTester {
  async test(plugin: Plugin): Promise<TestReport> {
    const tests = this.discoverTests(plugin);
    const results: TestResult[] = [];

    for (const test of tests) {
      const result = await this.runTest(plugin, test);
      results.push(result);
    }

    return {
      total: tests.length,
      passed: results.filter(r => r.passed).length,
      failed: results.filter(r => !r.passed).length,
      results
    };
  }

  private async runTest(
    plugin: Plugin,
    test: TestCase
  ): Promise<TestResult> {
    try {
      // 1. 设置测试环境
      const context = await this.setupTestContext(plugin);

      // 2. 运行测试
      await test.run(context);

      // 3. 清理
      await this.cleanup(context);

      return { passed: true, test };
    } catch (error) {
      return {
        passed: false,
        test,
        error: error.message
      };
    }
  }

  private setupTestContext(plugin: Plugin): Promise<TestContext> {
    // 创建隔离的测试环境
    return {
      mockFS: this.createMockFS(),
      mockHTTP: this.createMockHTTP(),
      mockEvents: this.createMockEvents(),
      ...this.createSandbox(plugin)
    };
  }
}
```

### 插件开发工具

#### 1. CLI 工具

```bash
# 创建新插件
$ claude plugin create my-plugin

# 验证插件
$ claude plugin validate ./my-plugin

# 打包插件
$ claude plugin build ./my-plugin

# 发布插件
$ claude plugin publish ./my-plugin
```

#### 2. 调试支持

```typescript
class PluginDebugger {
  async debug(plugin: Plugin): Promise<void> {
    // 1. 启用调试模式
    plugin.debugMode = true;

    // 2. 注入调试器
    this.injectDebugger(plugin);

    // 3. 启动监控
    this.startMonitoring(plugin);

    // 4. 等待调试连接
    await this.waitForDebugConnection();

    console.log('Debug mode enabled. Attach your debugger...');
  }

  private injectDebugger(plugin: Plugin): void {
    // 在插件代码中插入断点
    const files = glob('**/*.{js,ts}', { cwd: plugin.dir });
    for (const file of files) {
      this.instrumentFile(file);
    }
  }
}
```

---

## 📚 实战案例：开发一个日志分析插件

### 需求分析

- 📊 读取应用日志文件
- 🔍 分析错误和警告
- 📈 生成统计报告
- 💡 提供优化建议

### 实现代码

#### 1. plugin.json

```json
{
  "name": "log-analyzer",
  "version": "1.0.0",
  "description": "Analyze application logs",
  "author": "Your Name",
  "type": "plugin",
  "exports": {
    "commands": ["./commands/*.md"],
    "skills": ["./skills/*"]
  },
  "permissions": [
    "file:read"
  ]
}
```

#### 2. 命令文件 (commands/analyze.md)

```markdown
---
name: "analyze-logs"
description: "Analyze application logs"
---

分析日志文件...

{{file:read ./logs/app.log}}

## 分析结果

### 错误统计
- 总错误数: {{errorCount}}
- 最常见错误: {{mostCommonError}}

### 警告统计
- 总警告数: {{warningCount}}

### 性能指标
- 平均响应时间: {{avgResponseTime}}ms
- P95 响应时间: {{p95ResponseTime}}ms

### 优化建议
{{suggestions}}
```

#### 3. 技能文件 (skills/analyzer/SKILL.md)

```markdown
---
name: "log-analysis"
description: "Analyze log files and generate reports"
triggers:
  - "analyze logs"
  - "check errors"
  - "review warnings"
---

# 日志分析技能

## 使用场景
当用户需要分析应用日志时自动触发

## 分析流程

### 1. 读取日志
```typescript
const logs = await readFile(path, 'utf-8');
const lines = logs.split('\n');
```

### 2. 解析日志
```typescript
const entries = lines.map(line => parseLogLine(line));
const errors = entries.filter(e => e.level === 'ERROR');
const warnings = entries.filter(e => e.level === 'WARN');
```

### 3. 统计分析
```typescript
const errorCount = errors.length;
const errorTypes = groupBy(errors, e => e.type);
const mostCommonError = maxBy(Object.entries(errorTypes), ([, count]) => count);
```

### 4. 生成报告
```markdown
## 日志分析报告

### 时间范围
{{startTime}} - {{endTime}}

### 错误汇总
| 类型 | 次数 | 占比 |
|------|------|------|
{{errorTypes}}

### 性能指标
- 平均响应时间: {{avgTime}}ms
- P95 响应时间: {{p95Time}}ms
- P99 响应时间: {{p99Time}}ms

### 建议
{{recommendations}}
```
```

---

## ✅ 章节总结

### 入门级要点
- ✅ 理解插件是什么
- ✅ 掌握插件的基本使用方法
- ✅ 了解官方插件示例

### 中级要点
- ✅ 掌握插件开发流程
- ✅ 理解插件元数据
- ✅ 掌握组件导出方式
- ✅ 了解依赖管理

### 专家级要点
- ✅ 深入插件系统架构
- ✅ 掌握沙箱隔离机制
- ✅ 理解插件通信方式
- ✅ 掌握安全策略
- ✅ 了解市场集成

---

**下一步：** 学习 [03 - 命令系统](./03-command-system.md) 🚀
