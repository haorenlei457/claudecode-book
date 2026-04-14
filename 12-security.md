# 12 - 安全机制

## 📋 模块介绍

安全机制保护 Claude Code 的操作安全，防止意外或恶意操作。本章将讲解安全特性和最佳实践。

---

## 🟢 入门级：安全基础

### 基本安全特性

```markdown
✅ 权限控制 - 文件读写权限
✅ 操作确认 - 危险操作需确认
✅ 路径验证 - 防止路径遍历
✅ 输入验证 - 防止注入攻击
```

### 安全检查

```bash
# 文件写入检查
claude> 写入到 /etc/hosts

⚠️  警告：系统文件，需要确认

# 危险命令检查
claude> 运行 rm -rf /

❌ 拒绝：危险命令
```

---

## 🟡 中级：安全配置

### 权限管理

```json
{
  "permissions": {
    "file:read": true,
    "file:write": ["./src/**", "./tests/**"],
    "bash:run": true,
    "git:read": true,
    "git:write": true
  },
  "security": {
    "blockDangerousCommands": true,
    "requireConfirmation": true,
    "allowedPaths": ["./src", "./tests"]
  }
}
```

### 安全钩子

```bash
#!/bin/bash
# 检查敏感信息

if grep -i "password\|secret\|api_key" "$1"; then
  echo "❌ 发现敏感信息"
  exit 1
fi

exit 0
```

---

## 🔴 专家级：安全机制深度剖析

### 权限管理器

```typescript
class PermissionManager {
  private permissions: Map<string, Permission[]>;

  check(
    agentId: string,
    action: string,
    resource: string
  ): boolean {
    const perms = this.permissions.get(agentId) || [];

    for (const perm of perms) {
      if (perm.action === action) {
        return this.checkResource(perm, resource);
      }
    }

    return false;
  }

  private checkResource(perm: Permission, resource: string): boolean {
    for (const pattern of perm.allowedPatterns) {
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
    // 移除危险字符
    let sanitized = input.replace(/[;&|`$()]/g, '');

    // 路径遍历防护
    sanitized = sanitized.replace(/\.\.[/\\]/g, '');

    return sanitized;
  }

  validatePath(path: string): boolean {
    const normalized = path.normalize(path);
    const cwd = process.cwd();

    return normalized.startsWith(cwd);
  }
}
```

### 审计日志

```typescript
class AuditLogger {
  async log(event: AuditEvent): Promise<void> {
    const log = {
      timestamp: Date.now(),
      agentId: event.agentId,
      action: event.action,
      resource: event.resource,
      result: event.result
    };

    await fs.appendFile(
      this.logFile,
      JSON.stringify(log) + '\n'
    );
  }

  async query(filters: QueryFilters): Promise<AuditEvent[]> {
    const logs = await this.readLogs();
    return logs.filter(log => this.matches(log, filters));
  }
}
```

---

## ✅ 章节总结

### 入门级要点
- ✅ 理解基本安全特性
- ✅ 掌握安全检查

### 中级要点
- ✅ 掌握权限管理
- ✅ 理解安全配置

### 专家级要点
- ✅ 深入权限管理实现
- ✅ 掌握输入验证
- ✅ 理解审计日志

---

## 🎉 恭喜完成所有模块！

你已经完成了 Claude Code 的深度学习！现在你可以：
- ✅ 熟练使用 Claude Code 的所有功能
- ✅ 深入理解各个模块的工作原理
- ✅ 开发自定义插件、命令、代理和技能
- ✅ 修改和扩展 Claude Code 源码
- ✅ 贡献到 Claude Code 社区

继续探索，成为 Claude Code 专家！🚀
