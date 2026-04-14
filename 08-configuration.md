# 08 - 配置系统

## 📋 模块介绍

配置系统管理 Claude Code 的所有设置，包括全局配置、项目配置和用户偏好。本章将讲解配置的管理和使用。

---

## 🟢 入门级：配置基础认知

### 配置文件位置

```
全局配置：
~/.claude/settings.json
~/.claude/CLAUDE.md

项目配置：
.claude/settings.json
.claude/CLAUDE.md
```

### 配置优先级

```
项目配置 > 全局配置 > 默认配置
```

### 常用配置项

| 配置项 | 说明 | 默认值 |
|--------|------|--------|
| `model` | AI模型 | claude-sonnet-4 |
| `maxTokens` | 最大token数 | 4096 |
| `temperature` | 温度参数 | 0.7 |
| `plugins` | 插件列表 | [] |
| `hooks` | 钩子配置 | {} |

---

## 🟡 中级：配置管理

### 全局配置

```json
{
  "model": "claude-sonnet-4",
  "maxTokens": 8192,
  "temperature": 0.5,
  "plugins": [
    "code-review",
    "commit-commands"
  ],
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write",
        "hooks": ["~/.claude/hooks/pre-write.sh"]
      }
    ]
  }
}
```

### 项目配置

```json
{
  "model": "claude-opus-4",
  "maxTokens": 16384,
  "plugins": [
    "~/.claude/plugins/my-plugin"
  ],
  "memory": {
    "enabled": true,
    "maxSize": 1000
  }
}
```

### CLAUDE.md 配置

```markdown
# 项目配置

## 编码规范
- 遵循PEP 8
- 使用类型注解
- 添加文档字符串

## Git 工作流
- 特性分支：feature/xxx
- 提交格式：Conventional Commits
- PR需要审查

## 测试要求
- 单元测试覆盖率 > 80%
- 所有PR必须通过CI
```

---

## 🔴 专家级：配置系统深度剖析

### 配置加载器

```typescript
class ConfigLoader {
  async load(cwd: string): Promise<Config> {
    // 1. 加载默认配置
    const config = this.getDefaultConfig();

    // 2. 加载全局配置
    const globalConfig = await this.loadGlobalConfig();
    this.merge(config, globalConfig);

    // 3. 加载项目配置
    const projectConfig = await this.loadProjectConfig(cwd);
    this.merge(config, projectConfig);

    // 4. 验证配置
    this.validate(config);

    return config;
  }

  private merge(target: any, source: any): void {
    Object.assign(target, source);
  }

  private validate(config: Config): void {
    // 验证配置项
    if (!config.model) {
      throw new Error('Model is required');
    }
  }
}
```

---

## ✅ 章节总结

### 入门级要点
- ✅ 理解配置文件位置
- ✅ 掌握配置优先级
- ✅ 了解常用配置项

### 中级要点
- ✅ 掌握全局和项目配置
- ✅ 理解CLAUDE.md配置

### 专家级要点
- ✅ 深入配置加载机制
- ✅ 掌握配置验证

---

**下一步：** 学习 [09 - 文件操作与上下文管理](./09-file-context.md) 🚀
