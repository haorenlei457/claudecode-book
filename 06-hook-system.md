# 06 - 钩子系统（Hooks）

## 📋 模块介绍

钩子系统是 Claude Code 的事件驱动自动化机制，允许在特定事件发生时自动执行自定义逻辑。本章将讲解钩子的开发和使用。

---

## 🟢 入门级：钩子基础认知

### 什么是钩子？

钩子（Hook）是**事件触发器**，在特定事件发生时自动执行脚本。

**类比理解**：
- 🎣 像钓鱼钩，"钩住"特定事件
- 📱 像手机通知，事件发生时提醒
- 🔄 像Webhooks，HTTP请求触发回调

### 钩子能做什么？

```markdown
✅ 前置检查 - 在操作前验证条件
✅ 后置处理 - 在操作后执行清理
✅ 自动化 - 自动执行重复任务
✅ 监控 - 记录和追踪操作
✅ 防护 - 阻止危险操作
```

### 常用钩子类型

| 钩子事件 | 触发时机 | 用途 |
|----------|----------|------|
| **SessionStart** | 会话开始 | 初始化、加载配置 |
| **PreToolUse** | 工具使用前 | 验证权限、记录日志 |
| **PostToolUse** | 工具使用后 | 清理、更新状态 |
| **PreResponse** | 响应前 | 审查输出、格式化 |
| **Stop** | 停止时 | 保存状态、通知 |

### 钩子示例

```bash
# 示例1：文件写入前检查
事件：PreToolUse
匹配器：Write
脚本：检查是否写入受保护文件

# 示例2：测试运行后通知
事件：PostToolUse
匹配器：Bash
脚本：发送测试结果通知

# 示例3：会话开始时加载配置
事件：SessionStart
脚本：加载项目配置
```

---

## 🟡 中级：钩子开发与使用

### 钩子配置格式

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "*",
        "hooks": ["~/.claude/hooks/init.sh"]
      }
    ],
    "PreToolUse": [
      {
        "matcher": "Write",
        "hooks": [".claude/hooks/pre-write.sh"]
      },
      {
        "matcher": "Delete",
        "hooks": [".claude/hooks/pre-delete.sh"]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Bash",
        "hooks": [".claude/hooks/post-test.sh"]
      }
    ]
  }
}
```

### 钩子脚本格式

```bash
#!/bin/bash
# .claude/hooks/pre-write.sh

# 环境变量
# CLAUDE_HOOK_EVENT - 事件类型
# CLAUDE_TOOL_NAME - 工具名称
# CLAUDE_TOOL_PATH - 文件路径
# CLAUDE_TOOL_ARGS - 工具参数

FILE_PATH="${CLAUDE_TOOL_PATH}"

# 检查受保护的文件
PROTECTED_FILES=(
  "config/production.json"
  ".env.production"
  "credentials.json"
)

for protected in "${PROTECTED_FILES[@]}"; do
  if [[ "$FILE_PATH" == *"$protected"* ]]; then
    echo "⚠️  警告：尝试写入受保护的文件: $FILE_PATH"
    echo "请确认是否继续"
    exit 1
  fi
done

exit 0
```

### 创建钩子

#### 步骤1：创建钩子目录

```bash
mkdir -p .claude/hooks
```

#### 步骤2：编写钩子脚本

创建 `.claude/hooks/pre-commit.sh`：

```bash
#!/bin/bash
# 在Git提交前运行测试

echo "🧪 运行测试..."

# 运行测试
npm test

if [ $? -ne 0 ]; then
  echo "❌ 测试失败，阻止提交"
  exit 1
fi

echo "✅ 测试通过"
exit 0
```

#### 步骤3：添加执行权限

```bash
chmod +x .claude/hooks/pre-commit.sh
```

#### 步骤4：配置钩子

更新 `.claude/settings.json`：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "GitCommit",
        "hooks": [".claude/hooks/pre-commit.sh"]
      }
    ]
  }
}
```

#### 步骤5：测试钩子

```bash
$ claude
claude> /commit 添加新功能

🧪 运行测试...
✅ 测试通过

创建提交...
```

---

## 🔴 专家级：钩子系统深度剖析

### 钩子引擎实现

```typescript
class HookEngine {
  private hooks: Map<HookEvent, Hook[]>;

  async emit(event: HookEvent, context: HookContext): Promise<void> {
    const hooks = this.hooks.get(event) || [];

    for (const hook of hooks) {
      // 检查匹配器
      if (this.matches(hook.matcher, context)) {
        try {
          // 执行钩子脚本
          await this.executeHook(hook, context);
        } catch (error) {
          console.error(`Hook execution failed: ${error}`);

          // 阻止后续操作（如果钩子失败）
          if (hook.blockOnError) {
            throw new Error('Hook blocked operation');
          }
        }
      }
    }
  }

  private matches(matcher: string, context: HookContext): boolean {
    // 通配符匹配
    if (matcher === '*') return true;

    // 精确匹配
    if (matcher === context.toolName) return true;

    // 正则表达式匹配
    const regex = new RegExp(matcher);
    return regex.test(context.toolName);
  }

  private async executeHook(
    hook: Hook,
    context: HookContext
  ): Promise<void> {
    // 设置环境变量
    const env = {
      ...process.env,
      CLAUDE_HOOK_EVENT: hook.event,
      CLAUDE_TOOL_NAME: context.toolName,
      CLAUDE_TOOL_PATH: context.path,
      CLAUDE_TOOL_ARGS: JSON.stringify(context.args)
    };

    // 执行脚本
    const result = spawn('bash', [hook.path], {
      env,
      cwd: context.workingDir
    });

    await new Promise((resolve, reject) => {
      result.on('close', (code) => {
        if (code === 0) {
          resolve();
        } else {
          reject(new Error(`Hook exited with code ${code}`));
        }
      });
    });
  }
}
```

### 钩子链执行

```typescript
class HookChain {
  async execute(
    hooks: Hook[],
    context: HookContext
  ): Promise<HookResult[]> {
    const results: HookResult[] = [];

    for (const hook of hooks) {
      if (!this.matches(hook.matcher, context)) {
        continue;
      }

      const result = await this.executeHook(hook, context);
      results.push(result);

      // 阻断链式执行
      if (result.blocked) {
        break;
      }
    }

    return results;
  }

  private async executeHook(
    hook: Hook,
    context: HookContext
  ): Promise<HookResult> {
    const start = Date.now();

    try {
      await this.runScript(hook.path, context);

      return {
        hook,
        success: true,
        duration: Date.now() - start
      };
    } catch (error) {
      return {
        hook,
        success: false,
        duration: Date.now() - start,
        error: error.message,
        blocked: hook.blockOnError
      };
    }
  }
}
```

---

## 📚 实战案例：开发安全防护钩子

### 需求
- 🔒 防止提交敏感信息
- 🛡️ 阻止危险命令执行
- 📊 记录所有操作

### 实现

#### 1. 敏感信息检查

```bash
#!/bin/bash
# .claude/hooks/check-secrets.sh

# 检查提交中是否包含敏感信息

FILES=$(git diff --cached --name-only)

for FILE in $FILES; do
  # 检查API密钥
  if git diff --cached "$FILE" | grep -i "api[_-]key\|secret\|password"; then
    echo "❌ 发现敏感信息: $FILE"
    exit 1
  fi

  # 检查私钥
  if git diff --cached "$FILE" | grep "BEGIN.*PRIVATE"; then
    echo "❌ 发现私钥: $FILE"
    exit 1
  fi
done

echo "✅ 未发现敏感信息"
exit 0
```

#### 2. 危险命令拦截

```bash
#!/bin/bash
# .claude/hooks/check-dangerous.sh

DANGEROUS_COMMANDS=(
  "rm -rf"
  "dd if="
  ":(){:|:&};:"
  "mkfs"
)

for cmd in "${DANGEROUS_COMMANDS[@]}"; do
  if echo "$CLAUDE_COMMAND" | grep -q "$cmd"; then
    echo "⚠️  检测到危险命令: $cmd"
    echo "请确认是否继续"
    exit 1
  fi
done

exit 0
```

---

## ✅ 章节总结

### 入门级要点
- ✅ 理解钩子的概念
- ✅ 掌握基本使用方法
- ✅ 了解常用钩子类型

### 中级要点
- ✅ 掌握钩子配置格式
- ✅ 理解钩子脚本编写
- ✅ 学会创建自定义钩子

### 专家级要点
- ✅ 深入钩子引擎实现
- ✅ 掌握钩子链执行
- ✅ 理解性能优化

---

**下一步：** 学习 [07 - MCP协议](./07-mcp-protocol.md) 🚀
