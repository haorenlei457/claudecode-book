# 12 - 安全机制

## 📋 模块介绍

安全是 Claude Code 的核心设计理念之一。本章将深入讲解安全机制，帮助你理解如何保护代码、数据和系统安全。

---

## 🟢 入门级：安全基础

### 为什么安全很重要？

```markdown
⚠️  潜在风险：
- 意外删除重要文件
- 泄露敏感信息（API密钥、密码）
- 执行恶意代码
- 权限提升攻击

✅ Claude Code 的安全保障：
- 文件系统隔离
- 命令执行限制
- 敏感信息检测
- 操作确认机制
```

### 基本安全特性

| 特性 | 说明 | 示例 |
|------|------|------|
| **路径验证** | 防止目录遍历 | 拒绝 `../../../etc/passwd` |
| **命令过滤** | 阻止危险命令 | 拒绝 `rm -rf /` |
| **敏感信息检测** | 防止密钥泄露 | 检测 `API_KEY=xxx` |
| **权限控制** | 文件操作限制 | 只能写入项目目录 |
| **确认机制** | 危险操作确认 | 删除前询问 |

### 安全检查示例

```bash
# 尝试访问系统文件
claude> 读取 /etc/passwd
⚠️  警告：访问系统文件
❌ 操作被拒绝

# 尝试危险命令
claude> 运行 rm -rf /
❌ 危险命令已阻止

# 尝试写入敏感信息
claude> 在代码中添加 API_KEY="sk-123456"
⚠️  警告：检测到敏感信息
建议：使用环境变量
```

---

## 🟡 中级：安全配置与最佳实践

### 权限管理

```json
{
  "permissions": {
    "file:read": true,
    "file:write": [
      "./src/**",
      "./tests/**",
      "./docs/**"
    ],
    "file:delete": [
      "./temp/**",
      "./build/**"
    ],
    "bash:run": true,
    "bash:allowedCommands": [
      "npm",
      "yarn",
      "git",
      "python",
      "pytest"
    ],
    "git:read": true,
    "git:write": true
  }
}
```

### 安全钩子

```bash
#!/bin/bash
# pre-commit-security.sh

# 检查敏感信息
FILES=$(git diff --cached --name-only)

for FILE in $FILES; do
  # 检查API密钥
  if git diff --cached "$FILE" | grep -Ei "(api[_-]?key|secret|password|token)\s*[=:]\s*['\"][^'\"]+['\"]"; then
    echo "❌ $FILE 包含硬编码的敏感信息"
    exit 1
  fi

  # 检查私钥
  if git diff --cached "$FILE" | grep -E "BEGIN.*PRIVATE KEY"; then
    echo "❌ $FILE 包含私钥"
    exit 1
  fi

  # 检查调试代码
  if git diff --cached "$FILE" | grep -E "(console\.log|debugger|print\()"; then
    echo "⚠️  $FILE 包含调试代码"
  fi
done

echo "✅ 安全检查通过"
exit 0
```

### 安全配置最佳实践

#### 1. 最小权限原则

```json
{
  "permissions": {
    "file:write": [
      "./src/**",
      "./tests/**"
    ],
    "bash:run": false,
    "bash:allowedCommands": []
  }
}
```

#### 2. 敏感信息保护

```bash
# .env.example
DATABASE_URL=postgresql://user:pass@localhost/db
API_KEY=your_api_key_here
SECRET_KEY=your_secret_key_here

# 添加到 .gitignore
echo ".env" >> .gitignore
echo "*.key" >> .gitignore
echo "*.pem" >> .gitignore
```

---

## 🔴 专家级：安全机制深度剖析

### 权限管理器

```typescript
class PermissionManager {
  private permissions: Map<string, Permission> = new Map();

  check(
    action: string,
    resource: string,
    context: SecurityContext
  ): PermissionResult {
    const permission = this.permissions.get(action);

    if (!permission) {
      return { allowed: false, reason: 'Permission not defined' };
    }

    // 检查资源匹配
    if (!this.matchesResource(permission.allowedResources, resource)) {
      return { allowed: false, reason: 'Resource not allowed' };
    }

    // 检查上下文条件
    if (!this.checkConditions(permission.conditions, context)) {
      return { allowed: false, reason: 'Conditions not met' };
    }

    return { allowed: true };
  }

  private matchesResource(
    patterns: string[],
    resource: string
  ): boolean {
    for (const pattern of patterns) {
      if (minimatch(resource, pattern)) {
        return true;
      }
    }
    return false;
  }
}
```

### 输入验证器

```typescript
class InputValidator {
  sanitize(input: string): string {
    // 防止命令注入
    let sanitized = input.replace(/[;&|`$()]/g, '');

    // 防止路径遍历
    sanitized = sanitized.replace(/\.\.[/\\]/g, '');

    return sanitized;
  }

  validatePath(path: string, allowedPaths: string[]): boolean {
    const normalized = path.normalize(path);

    for (const allowed of allowedPaths) {
      if (normalized.startsWith(allowed)) {
        return true;
      }
    }

    return false;
  }
}
```

### 审计日志

```typescript
class AuditLogger {
  async log(event: AuditEvent): Promise<void> {
    const logEntry = {
      timestamp: Date.now(),
      user: event.userId,
      action: event.action,
      resource: event.resource,
      result: event.result,
      details: event.details
    };

    await this.store(logEntry);
  }

  async query(filters: AuditFilters): Promise<AuditEvent[]> {
    // 查询审计日志
    return this.database.query('audit_logs', filters);
  }
}
```

---

## ✅ 章节总结

### 入门级要点
- ✅ 理解安全重要性
- ✅ 掌握基本安全特性
- ✅ 了解安全检查

### 中级要点
- ✅ 掌握权限管理
- ✅ 理解安全钩子
- ✅ 掌握最佳实践

### 专家级要点
- ✅ 深入权限管理实现
- ✅ 掌握输入验证
- ✅ 理解审计日志

---

## 🎉 恭喜完成所有模块！

你已经完成了 Claude Code 的深度学习！现在你可以：
- ✅ 熟练使用所有功能
- ✅ 深入理解工作原理
- ✅ 开发自定义扩展
- ✅ 修改和优化源码

继续探索，成为 Claude Code 专家！🚀
