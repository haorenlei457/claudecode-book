# 08 - 配置系统

## 📋 模块介绍

配置系统管理 Claude Code 的所有设置，包括全局配置、项目配置和用户偏好。本章将详细讲解配置的层级、加载机制和使用技巧。

---

## 🟢 入门级：配置基础认知

### 🤔 什么是配置？

#### 简单理解

**配置就像Claude Code的"设置"**，可以控制它的各种行为和功能。

**类比理解**：
```
设置手机：
- 壁音/震动/亮度/壁纸
- 蓱息/勿扰/待机时间
- 桌件权限/存储空间

Claude Code 配置：
- AI模型选择
- 最大Token数
- 插件列表
- 文件读写权限
- 终端执行权限
- Git 操作权限
```

---

### 📦 配置层级

```mermaid
graph TD
    A[用户] -->|优先级1| B[全局配置]
    A -->|项目配置| C|优先级2| B
    A -->|项目子配置| D|优先级3| C
    A -->|默认配置| E|优先级4| D
    
    style A fill:#e1f5e3
    style B fill:#FFE5E3
    style C fill:#FFD43D
    style D fill:#FFD43D
    style E fill:#FFF0F0
```

### 配置优先级

```
1. 当前目录CLAUDE.md > 父目录CLAUDE.md > 全局CLAUDE.md
2. 全局CLAUDE.md > 默认配置
```

---

### 🎯 常用配置项

#### 1. AI模型配置

```json
{
  "model": "claude-sonnet-4",
  "maxTokens": 8192,
  "temperature": 0.7
}
```

**配置说明**：
- `model`: 模型选择（不同模型能力不同）
- `maxTokens`: 最大token数（影响回复长度）
- `temperature`: 创意度（0.0-1.0，越高越有创意）

#### 2. 插件列表

```json
{
  "plugins": [
    "code-review",
    "commit-commands",
    "feature-dev",
    "security-guidance"
  ]
}
```

**配置说明**：
- `plugins`: 启用的插件列表
- 顺序很重要：后面插件的会覆盖前面的
- 全局插件：所有项目都会使用
- 项目插件：只在该项目有效

#### 3. 钩子配置

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write",
        "hooks": [".claude/hooks/pre-write.sh"]
      }
    ]
  }
}
```

#### 4. 权限控制

```json
{
  "permissions": {
    "file:read": true,
    "file:write": ["./src/**", "./tests/**"],
    "bash:run": true
  }
}
```

---

## 🟡 中级：配置管理与进阶用法

### 🔧 全局配置详解

#### 1. 设置个人偏好

编辑 `~/.claude/CLAUDE.md`：

```markdown
# 个人偏好配置

## 编码风格
- 偏好简洁的代码
- 使用类型注解
- 添加必要注释

## 沟通方式
- 直接对话告诉Claude
- 逐步建立个人偏好

## 学习模式
- 喜欢详细解释
- 生成完整示例
- 边学边练

## 工具选择
- 优先使用内置工具
- 谨慎使用外部命令
```

---

### 🔧 项目级配置详解

#### 1. 项目类型检测

```json
{
  "projectType": {
    "typescript": {
      "framework": "React",
      "state": "16"
    },
    "python": {
      "framework": "Django",
      "version": "4.2"
    },
    "go": {
      "framework": "Gin",
      "version": "1.21"
    }
  }
}
```

#### 2. Git 工作流配置

```json
{
  "git": {
    "workflow": "feature-branch",
    "commit": {
      "format": "conventional",
      "enforce": true
    }
  }
}
```

#### 3. 测试配置

```json
{
  "test": {
    "framework": "jest",
    "command": "jest",
    "coverage": 80
  }
}
```

---

## 🔴 专家级：配置系统深度剖析

### ⚙️ 配置加载算法

```typescript
class ConfigLoader {
  async load(cwd: string): Promise<Config> {
    // 1. 从默认配置开始
    let config = this.getDefaultConfig();
    
    // 2. 加载全局配置
    const globalConfig = await this.loadGlobalConfig();
    config = this.merge(config, globalConfig);
    
    // 3. 加载项目配置
    const projectConfig = await this.loadProjectConfig(cwd);
    config = this.merge(config, projectConfig);
    
    // 4. 加载环境变量覆盖
    config = this.applyEnvOverrides(config);
    
    // 5. 验证配置
    this.validate(config);
    
    return config;
  }
  
  private merge(target: any, source: any): any {
    for (const key of Object.keys(source)) {
      if (source[key] && typeof source[key] === 'object' && !Array.isArray(source[key])) {
        target[key] = { ...target[key], ...source[key] };
      } else if (Array.isArray(source[key])) {
        target[key] = [...(target[key] || []), ...source[key]];
      } else {
        target[key] = source[key];
      }
    }
    return target;
  }
}
```

### 🔐 动态配置更新

```typescript
class ConfigWatcher {
  async watch(configPath: string): Promise<void> {
    // 监听配置文件变化
    const watcher = chokidar.watch(configPath, {
      ignoreInitial: false,
      awaitWrite: true
    });
    
    watcher.on('change', async (path) => {
      console.log(`配置文件已更新: ${path}`);
      const newConfig = await this.load(configPath);
      this.config = newConfig;
      this.notifyChange(newConfig);
    });
  }
  
  async notifyChange(newConfig: Config): void {
    console.log('配置已更新');
    // 通知其他组件重新加载
  }
}
```

### 🔐 性能优化策略

```typescript
class ConfigOptimizer {
  private cache: Map<string, CachedConfig> = new Map();
  
  async get<T>(key: string, factory: () => Promise<T>): Promise<T> {
    // 检查缓存
    const cached = this.cache.get(key);
    if (cached && !this.isExpired(cached)) {
      return cached.value;
    }
    
    // 计算并缓存
    const value = await factory();
    this.cache.set(key, {
      value,
      timestamp: Date.now(),
      ttl: 60000  // 60秒缓存
    });
    
    return value;
  }
}
```

---

## 📚 实战案例：配置管理系统

### 需求
创建一个智能配置管理系统，支持环境切换、项目切换、配置热更新。

### 实现

#### 1. 环境切换功能

```typescript
// config/env-switcher.ts
class ConfigManager {
  async switchEnvironment(env: 'development' | 'staging' | 'production'): Promise<void> {
    const envs = ['development', 'staging', 'production'];
    const currentIndex = envs.indexOf(env);
    const nextIndex = (currentIndex + 1) % envs.length;
    
    console.log(`切换环境: ${env} -> ${envs[nextIndex]}`);
    
    this.currentEnv = envs[nextIndex];
    await this.applyConfig(this.currentEnv);
  }
  
  async applyConfig(env: string): Promise<void> {
    const configs = {
      development: {
        model: 'claude-sonnet-4',
        maxTokens: 4096,
        temperature: 0.3
      },
      staging: {
        model: 'claude-sonnet-4',
        maxTokens: 8192,
        temperature: 0.5
      },
      production: {
        model: 'claude-opus-4',
        maxTokens: 16384,
        temperature: 0.7
      }
    };
    
    const config = configs[env as keyof typeof typeof configs];
    this.config = config;
    console.log(`应用配置: ${env}模式`);
  }
}
```

#### 2. 项目切换功能

```typescript
// config/project-switcher.ts
class ProjectConfigManager {
  async switchProject(projectName: string): Promise<void> {
    const projects = ['frontend', 'backend', 'shared'];
    
    if (!projects.includes(projectName)) {
      throw new Error(`Unknown project: ${projectName}`);
    }
    
    console.log(`切换到项目: ${projectName}`);
    await this.switchProjectContext(projectName);
  }
  
  private async switchProjectContext(projectName: string): Promise<void> {
    // 加载项目配置
    const projectPath = this.getProjectPath(projectName);
    const projectConfig = await this.loadProjectConfig(projectPath);
    
    // 应用项目配置
    this.config.project = projectConfig;
    console.log(`项目配置已应用: ${projectName}`);
  }
}
```

---

## ✅ 章节总结

### 入门级要点
- ✅ 理解配置的层级结构
- 掌握常用配置项
- 学会项目级配置

### 中级要点
- ✅ 掌握配置加载顺序
- 理解环境变量覆盖
- 学会Git工作流配置
- 学会测试配置

### 专家级要点
- ✅ 深入配置加载机制
- 掌握动态配置更新
- 掌握性能优化策略
- 理解配置管理系统

### 📊 相关图表

- **配置加载流程图**：展示从默认→全局→项目→环境的加载过程
- **配置优先级图**：展示不同配置的覆盖关系
- **动态配置更新流程图**：展示热更新机制

---

**下一步：** 学习 [09 - 文件操作与上下文管理](./09-file-context.md) 🚀
