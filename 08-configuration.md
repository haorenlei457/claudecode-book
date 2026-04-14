# 08 - 配置系统

## 📋 模块介绍

配置系统是 Claude Code 的"大脑"，负责管理所有设置和偏好。一个良好的配置能让 Claude Code 更好地理解你的项目，提供更精准的帮助。本章将深入讲解配置的管理、加载机制和使用技巧。

---

## 🟢 入门级：配置基础认知

### 什么是配置？

配置是**告诉 Claude Code 如何工作的指令集合**，包括：
- 🤖 使用哪个AI模型
- 🔌 加载哪些插件
- ⚙️ 启用哪些功能
- 📝 遵循什么规范

**类比理解**：
- 📱 像手机的设置菜单
- 🎮 像游戏的选项配置
- ⚙️ 像软件的首选项

### 配置文件位置

Claude Code 使用分层配置系统：

```
系统级配置（最低优先级）
    ↓
用户全局配置 ~/.claude/
    ↓
项目级配置 ./.claude/（最高优先级）
```

**具体位置**：

```bash
# 全局配置（影响所有项目）
~/.claude/settings.json          # 核心配置
~/.claude/CLAUDE.md              # 个人偏好和习惯
~/.claude/commands/              # 全局命令
~/.claude/agents/                # 全局代理
~/.claude/skills/                # 全局技能
~/.claude/hooks/                 # 全局钩子

# 项目配置（仅影响当前项目）
./.claude/settings.json          # 项目特定配置
./CLAUDE.md                      # 项目上下文
./src/CLAUDE.md                  # 子项目配置
./tests/CLAUDE.md                # 测试规则
```

### 配置优先级

当配置冲突时，按以下优先级覆盖：

```
子项目配置 > 项目根配置 > 全局配置 > 默认配置

示例：
- 如果在 src/CLAUDE.md 和 ./CLAUDE.md 中定义了不同的编码规范
- 在 src/ 目录下工作时，使用 src/CLAUDE.md 的配置
```

### 常用配置项

#### settings.json 核心配置

```json
{
  "model": "claude-sonnet-4",
  "maxTokens": 8192,
  "temperature": 0.7,
  "plugins": ["code-review", "commit-commands"],
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write",
        "hooks": [".claude/hooks/pre-write.sh"]
      }
    ]
  },
  "permissions": {
    "file:read": true,
    "file:write": ["./src/**", "./tests/**"],
    "bash:run": true
  }
}
```

**字段说明**：

| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| `model` | string | claude-sonnet-4 | AI模型版本 |
| `maxTokens` | number | 4096 | 最大响应token数 |
| `temperature` | number | 0.7 | 创意度(0-1) |
| `plugins` | array | [] | 启用的插件列表 |
| `hooks` | object | {} | 钩子配置 |
| `permissions` | object | {} | 权限设置 |

### 配置的作用

#### 1. 个性化AI行为

```json
{
  "model": "claude-opus-4",
  "temperature": 0.3,
  "maxTokens": 16384
}
```
- 使用更强的模型
- 更低的温度（更确定）
- 更长的响应

#### 2. 自动化工作流

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "*",
        "hooks": [".claude/hooks/init.sh"]
      }
    ]
  }
}
```
- 会话开始时自动加载环境

#### 3. 安全管控

```json
{
  "permissions": {
    "file:write": ["./src/**", "./docs/**"],
    "bash:run": false
  }
}
```
- 限制可写入的目录
- 禁止执行bash命令

---

## 🟡 中级：配置管理与进阶用法

### 全局配置详解

#### 1. 创建全局配置

```bash
# 创建配置目录
mkdir -p ~/.claude

# 创建配置文件
cat > ~/.claude/settings.json << 'EOF'
{
  "model": "claude-sonnet-4",
  "maxTokens": 8192,
  "temperature": 0.5,
  "plugins": [
    "code-review"
  ],
  "memory": {
    "enabled": true,
    "maxSize": 1000
  },
  "output": {
    "style": "concise",
    "includeTimestamps": true
  }
}
EOF
```

#### 2. 全局CLAUDE.md

```bash
cat > ~/.claude/CLAUDE.md << 'EOF'
# 个人偏好配置

## 编码风格
- 偏好简洁的代码
- 使用函数式编程风格
- 注重类型安全

## 沟通方式
- 直接、简洁
- 提供代码示例
- 解释"为什么"

## 常用技术栈
- TypeScript
- Python
- React
- Node.js
EOF
```

### 项目级配置详解

#### 1. Python项目配置示例

```json
{
  "model": "claude-sonnet-4",
  "maxTokens": 8192,
  "plugins": [],
  "python": {
    "version": "3.11",
    "formatter": "black",
    "linter": "pylint",
    "testRunner": "pytest"
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write",
        "hooks": [".claude/hooks/check-python-syntax.sh"]
      }
    ]
  }
}
```

#### 2. JavaScript项目配置示例

```json
{
  "model": "claude-sonnet-4",
  "plugins": ["commit-commands"],
  "javascript": {
    "runtime": "node",
    "version": "20",
    "packageManager": "npm",
    "testCommand": "npm test"
  },
  "eslint": {
    "enabled": true,
    "configFile": ".eslintrc.js"
  }
}
```

### CLAUDE.md 配置详解

#### 1. 项目级 CLAUDE.md

```markdown
# MyProject 项目配置

## 项目概述
- 类型：Web应用
- 技术栈：React + TypeScript + Node.js
- 数据库：PostgreSQL

## 编码规范

### 命名规范
- 组件：PascalCase (UserProfile.tsx)
- 函数：camelCase (getUserData)
- 常量：UPPER_SNAKE_CASE (MAX_RETRY_COUNT)
- 接口：I前缀 (IUserData)

### 文件组织
```
src/
├── components/     # 可复用组件
├── pages/         # 页面组件
├── hooks/         # 自定义hooks
├── utils/         # 工具函数
├── services/      # API服务
└── types/         # 类型定义
```

### 代码风格
- 使用 TypeScript 严格模式
- 所有函数必须有返回类型
- 复杂逻辑需要注释
- 使用 Prettier 格式化

## Git 工作流

### 分支策略
- main: 生产分支
- develop: 开发分支
- feature/*: 功能分支
- hotfix/*: 紧急修复

### 提交规范
```
<type>(<scope>): <subject>

<body>

<footer>
```

类型：
- feat: 新功能
- fix: 修复bug
- docs: 文档
- style: 格式
- refactor: 重构
- test: 测试
- chore: 构建/工具

## API规范
- RESTful API
- 使用 OpenAPI 3.0
- 统一返回格式
- 错误码规范

## 测试要求
- 单元测试覆盖率 > 80%
- 关键路径必须有集成测试
- 使用 Jest + React Testing Library
```

#### 2. 子项目级 CLAUDE.md（src/api/CLAUDE.md）

```markdown
# API层配置

## 技术栈
- Fastify
- TypeScript
- Prisma ORM

## 编码规范
- 使用依赖注入
- 控制器 + 服务分层
- 统一的错误处理
- 输入验证使用 Zod

## 数据库规范
- 表名：snake_case
- 字段：snake_case
- 外键：表名单数_id
- 时间戳：created_at, updated_at

## API设计
- 使用资源命名
- HTTP方法规范
- 状态码正确使用
- 统一错误响应格式
```

### 配置管理最佳实践

#### 1. 环境变量使用

```bash
# .env
CLAUDE_API_KEY=your_api_key
GITHUB_TOKEN=your_github_token
DATABASE_URL=postgresql://...
```

```json
// settings.json
{
  "apiKey": "${CLAUDE_API_KEY}",
  "mcpServers": {
    "github": {
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

#### 2. 配置验证

```bash
# 验证配置格式
claude --validate-config

# 检查配置问题
claude --check-config
```

#### 3. 配置备份与同步

```bash
# 备份配置
cp ~/.claude/settings.json ~/.claude/settings.json.backup

# 使用 Git 管理配置
cd ~/.claude && git init && git add . && git commit -m "Initial config"
```

---

## 🔴 专家级：配置系统深度剖析

### 配置加载机制

```typescript
interface ConfigLoader {
  load(cwd: string): Promise<Config>;
}

class DefaultConfigLoader implements ConfigLoader {
  private readonly defaultConfig: Config = {
    model: 'claude-sonnet-4',
    maxTokens: 4096,
    temperature: 0.7,
    plugins: [],
    hooks: {},
    permissions: {
      'file:read': true,
      'file:write': true,
      'bash:run': true
    }
  };

  async load(cwd: string): Promise<Config> {
    // 1. 从默认配置开始
    let config = { ...this.defaultConfig };

    // 2. 加载全局配置
    const globalConfig = await this.loadGlobalConfig();
    config = this.deepMerge(config, globalConfig);

    // 3. 加载项目配置
    const projectConfig = await this.loadProjectConfig(cwd);
    config = this.deepMerge(config, projectConfig);

    // 4. 加载环境变量覆盖
    config = this.applyEnvOverrides(config);

    // 5. 验证配置
    this.validate(config);

    return config;
  }

  private async loadGlobalConfig(): Promise<Partial<Config>> {
    const configPath = path.join(
      os.homedir(),
      '.claude',
      'settings.json'
    );

    if (await fs.pathExists(configPath)) {
      const content = await fs.readFile(configPath, 'utf-8');
      return JSON.parse(content);
    }

    return {};
  }

  private async loadProjectConfig(cwd: string): Promise<Partial<Config>> {
    const configPath = path.join(cwd, '.claude', 'settings.json');

    if (await fs.pathExists(configPath)) {
      const content = await fs.readFile(configPath, 'utf-8');
      return JSON.parse(content);
    }

    return {};
  }

  private deepMerge(target: any, source: any): any {
    const result = { ...target };

    for (const key of Object.keys(source)) {
      if (source[key] && typeof source[key] === 'object' && !Array.isArray(source[key])) {
        result[key] = this.deepMerge(target[key] || {}, source[key]);
      } else {
        result[key] = source[key];
      }
    }

    return result;
  }

  private applyEnvOverrides(config: Config): Config {
    const envMappings: Record<string, string> = {
      'CLAUDE_MODEL': 'model',
      'CLAUDE_MAX_TOKENS': 'maxTokens',
      'CLAUDE_TEMPERATURE': 'temperature'
    };

    for (const [envVar, configPath] of Object.entries(envMappings)) {
      const value = process.env[envVar];
      if (value) {
        this.setNestedValue(config, configPath, this.parseValue(value));
      }
    }

    return config;
  }

  private validate(config: Config): void {
    // 验证必需字段
    if (!config.model) {
      throw new ConfigError('Model is required');
    }

    // 验证数值范围
    if (config.temperature < 0 || config.temperature > 1) {
      throw new ConfigError('Temperature must be between 0 and 1');
    }

    if (config.maxTokens < 1 || config.maxTokens > 200000) {
      throw new ConfigError('Max tokens must be between 1 and 200000');
    }
  }
}
```

### 动态配置更新

```typescript
class ConfigWatcher {
  private watchers: Map<string, FSWatcher> = new Map();
  private config: Config;
  private callbacks: ConfigChangeCallback[] = [];

  async watch(configPaths: string[]): Promise<void> {
    for (const configPath of configPaths) {
      const watcher = chokidar.watch(configPath);

      watcher.on('change', async () => {
        console.log(`Config changed: ${configPath}`);

        // 重新加载配置
        const newConfig = await this.reloadConfig();

        // 计算差异
        const diff = this.computeDiff(this.config, newConfig);

        // 通知订阅者
        this.notifyChange(diff);

        this.config = newConfig;
      });

      this.watchers.set(configPath, watcher);
    }
  }

  onChange(callback: ConfigChangeCallback): void {
    this.callbacks.push(callback);
  }

  private notifyChange(diff: ConfigDiff): void {
    for (const callback of this.callbacks) {
      callback(diff);
    }
  }

  stop(): void {
    for (const watcher of this.watchers.values()) {
      watcher.close();
    }
  }
}
```

### 配置Schema验证

```typescript
import { z } from 'zod';

const ConfigSchema = z.object({
  model: z.string(),
  maxTokens: z.number().min(1).max(200000),
  temperature: z.number().min(0).max(1),
  plugins: z.array(z.string()),
  hooks: z.record(
    z.array(
      z.object({
        matcher: z.string(),
        hooks: z.array(z.string())
      })
    )
  ),
  permissions: z.object({
    'file:read': z.boolean().optional(),
    'file:write': z.union([z.boolean(), z.array(z.string())]).optional(),
    'bash:run': z.boolean().optional()
  }).optional()
});

type Config = z.infer<typeof ConfigSchema>;

function validateConfig(config: unknown): Config {
  return ConfigSchema.parse(config);
}
```

### 配置加密与安全

```typescript
class SecureConfig {
  private readonly algorithm = 'aes-256-gcm';
  private readonly keyLength = 32;

  async encryptConfig(
    config: Config,
    password: string
  ): Promise<EncryptedConfig> {
    const iv = crypto.randomBytes(16);
    const salt = crypto.randomBytes(32);

    const key = await this.deriveKey(password, salt);
    const cipher = crypto.createCipheriv(this.algorithm, key, iv);

    const plaintext = JSON.stringify(config);
    let encrypted = cipher.update(plaintext, 'utf8', 'hex');
    encrypted += cipher.final('hex');

    const authTag = cipher.getAuthTag();

    return {
      encrypted,
      iv: iv.toString('hex'),
      salt: salt.toString('hex'),
      authTag: authTag.toString('hex')
    };
  }

  async decryptConfig(
    encryptedConfig: EncryptedConfig,
    password: string
  ): Promise<Config> {
    const key = await this.deriveKey(
      password,
      Buffer.from(encryptedConfig.salt, 'hex')
    );

    const decipher = crypto.createDecipheriv(
      this.algorithm,
      key,
      Buffer.from(encryptedConfig.iv, 'hex')
    );

    decipher.setAuthTag(
      Buffer.from(encryptedConfig.authTag, 'hex')
    );

    let decrypted = decipher.update(
      encryptedConfig.encrypted,
      'hex',
      'utf8'
    );
    decrypted += decipher.final('utf8');

    return JSON.parse(decrypted);
  }

  private async deriveKey(password: string, salt: Buffer): Promise<Buffer> {
    return new Promise((resolve, reject) => {
      crypto.pbkdf2(
        password,
        salt,
        100000,
        this.keyLength,
        'sha256',
        (err, key) => {
          if (err) reject(err);
          else resolve(key);
        }
      );
    });
  }
}
```

---

## 📚 实战案例：配置管理系统

### 需求
- 🔧 管理多个项目的配置
- 🔄 自动同步配置
- 🔒 安全存储敏感信息
- 📊 配置审计追踪

### 实现

#### 1. 配置管理工具

```bash
#!/bin/bash
# claude-config-manager.sh

CONFIG_DIR="$HOME/.claude"
PROJECTS_DIR="$CONFIG_DIR/projects"

# 初始化项目配置
init_project() {
    local project_name=$1
    local project_path=$2

    mkdir -p "$PROJECTS_DIR/$project_name"

    cat > "$PROJECTS_DIR/$project_name/settings.json" << EOF
{
  "name": "$project_name",
  "path": "$project_path",
  "model": "claude-sonnet-4",
  "created": "$(date -Iseconds)"
}
EOF

    echo "✅ 项目配置已创建: $project_name"
}

# 同步配置
sync_config() {
    local project_name=$1
    local source="$PROJECTS_DIR/$project_name"
    local target=$(cat "$source/settings.json" | jq -r '.path')

    cp "$source/settings.json" "$target/.claude/settings.json"
    echo "✅ 配置已同步到: $target"
}

# 列出所有项目
list_projects() {
    for project in "$PROJECTS_DIR"/*/; do
        local name=$(basename "$project")
        local path=$(cat "$project/settings.json" | jq -r '.path')
        echo "- $name: $path"
    done
}

# 主命令
case "$1" in
    init)
        init_project "$2" "$3"
        ;;
    sync)
        sync_config "$2"
        ;;
    list)
        list_projects
        ;;
    *)
        echo "用法: $0 {init|sync|list}"
        ;;
esac
```

#### 2. 使用示例

```bash
# 初始化新项目
./claude-config-manager.sh init my-project /path/to/project

# 同步配置
./claude-config-manager.sh sync my-project

# 列出所有项目
./claude-config-manager.sh list
```

---

## ✅ 章节总结

### 入门级要点
- ✅ 理解配置的概念和作用
- ✅ 掌握配置文件位置
- ✅ 了解配置优先级
- ✅ 熟悉常用配置项

### 中级要点
- ✅ 掌握全局和项目配置
- ✅ 理解CLAUDE.md配置
- ✅ 掌握环境变量使用
- ✅ 了解配置验证方法

### 专家级要点
- ✅ 深入配置加载机制
- ✅ 掌握动态配置更新
- ✅ 理解Schema验证
- ✅ 掌握配置加密

---

**下一步：** 学习 [09 - 文件操作与上下文管理](./09-file-context.md) 🚀
