# 06 - 钩子系统（Hooks）

## 📋 模块介绍

钩子系统是 Claude Code 的"自动化助手"，让你可以在特定事件发生时自动执行自定义脚本。本章将详细讲解钩子的配置和使用方法。

---

## 🟢 入门级：钩子基础认知

### 🤔 什么是钩子？

#### 简单理解

**钩子（Hook）就像"自动触发器"**，在特定事件发生时自动执行脚本。

**类比理解**：

```
传统方式（手动操作）：
1. 提交前需要手动运行测试
2. 提交时需要手动检查代码
3. 发布前需要手动通知

使用钩子后（自动）：
1. 提交前：自动运行测试、检查代码
2. 提交后：自动通知团队
3. 发布后：自动通知状态
```

**核心优势**：
- ⚡ **自动化** - 无需手动执行
- 🔒 **及时** - 在关键节点触发
- 🔁 **可靠** - 避免人为遗漏
- 🔧 **灵活** - 支持任意脚本

---

### 🎯 钩子类型

| 事件类型 | 触发时机 | 用途 |
|----------|----------|------|
| **SessionStart** | 会话开始 | 初始化、加载配置 |
| **PreToolUse** | 工具使用前 | 验证权限、记录日志 |
| **PostToolUse** | 工具使用后 | 清理、更新状态 |
| **PreResponse** | 响应前 | 审查输出、格式化 |
| **Stop** | 停止时 | 保存状态、通知 |

---

### 🎯 钩子示例

#### 1. SessionStart 钩子（初始化）

```bash
#!/bin/bash
# .claude/hooks/init.sh

echo "🚀 初始化开发环境..."

# 检查环境变量
if [ ! -n "$CLAUDE_ENV" ]; then
    echo "⚠️  CLAUDE_ENV 未设置"
fi

# 加载项目配置
if [ -f CLAUDE.md ]; then
    echo "📋 发现 CLAUDE.md"
fi

echo "✅ 初始化完成"
```

**效果**：每次打开Claude Code时自动执行初始化

#### 2. PreToolUse 钩子（工具使用前检查）

```bash
#!/bin/bash
# .claude/hooks/pre-write.sh

FILE_PATH="${CLAUDE_TOOL_PATH}"

# 检查受保护的文件
PROTECTED_FILES=(
  "config/production.json"
  ".env.production"
  "credentials.json"
)

for protected in "${PROTECTED_FILES[@]}"; do
  if [[ "$FILE_PATH" == *"$protected"* ]]; then
    echo "⚠️️ 警告：尝试写入受保护的文件: $FILE_PATH"
    echo "❌ 操作被拒绝"
    exit 1
  fi
done

exit 0
```

**效果**：防止误写重要文件

#### 3. PostToolUse 钩子（工具使用后处理）

```bash
#!/bin/bash
# .claude/hooks/post-push.sh

echo "🚀 推送到远程仓库..."

# 执行推送
git push origin main

# 检查状态
STATUS=$(curl -s https://api.github.com/repos/haorenlei457/claudecode-book/commits/main/status)
if [ "$STATUS" = "\"pending\"" ]; then
    echo "⏳️ PR 正在审查中..."
else
    echo "✅ 推送成功"
fi
```

**效果**：推送后自动检查状态

---

## 🟡 中级：钩子开发与配置

### 🔧 钩子配置格式

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "*",
        "hooks": [".claude/hooks/init.sh"]
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
        "matcher": "GitCommit",
        "hooks": [".claude/hooks/post-commit.sh"]
      },
      {
        "matcher": "GitPush",
        "hooks": [".claude/hooks/post-push.sh"]
      }
    ]
  }
}
```

**字段说明**：

| 字段 | 说明 |
|------|------|
| `event` | 事件类型 |
| `matcher` | 匹配模式（*表示通配） |
| `hooks` | 要执行的钩子脚本 |

**匹配器模式**：

| 模式 | 说明 | 示例 |
|------|------|------|
| `"*"` | 匹配所有 | 匹配所有ToolUse事件 |
| `"Write"` | 匹配写文件操作 | 匹配所有Write操作 |
| `"Delete"` | 匹配删除操作 | 匹配所有Delete操作 |
| `"GitCommit"` | 匹配Git提交 | 匹配所有Git提交 |
| 正则表达式 | 高级匹配 | 匹配特定模式 |

---

### 🔧 钩子脚本编写

#### 钩子脚本结构

```bash
#!/bin/bash
# .claude/hooks/example-hook.sh

# 环境变量
# CLAUDE_HOOK_EVENT: 事件类型
# CLAUDE_TOOL_NAME: 工具名称
# CLAUDE_TOOL_PATH: 文件路径
# CLAUDE_TOOL_ARGS: 工具参数

# 获取环境变量
EVENT="${CLAUDE_HOOK_EVENT}"
TOOL="${CLAUDE_TOOL_NAME}"
PATH="${CLAUDE_TOOL_PATH}"
ARGS="${CLAUDE_TOOL_ARGS}"

# 检查工具
case "$TOOL" in
  Write)
    if [[ "$PATH" == *"config"* ]]; then
      echo "⚠️️ 警告：修改配置文件"
      echo "请确认是否继续"
      exit 1
      ;;
  fi
      ;;
  
  Bash)
    if [[ "$ARGS" == *"rm -rf"* ]]; then
      echo "⚠️️ 危险命令：$ARGS"
      echo "请确认是否继续"
      exit 1
      ;;
      ;;
esac
    ;;
esac
```

# 默认通过
exit 0
```

**编写技巧**：
- 使用 `esac` 处理错误
- 使用 `exit 0` 表示通过
- 使用 `exit 1` 表示失败

---

## 🔴 专家级：钩子系统深度剖析

### ⚙️ 钩子引擎实现

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
    try {
      const regex = new RegExp(matcher);
      return regex.test(context.toolName);
    } catch {
      return false;
    }
  }
  
  private async executeHook(hook: Hook, context: HookContext): Promise<void> {
    const env = {
      ...process.env,
      CLAUDE_HOOK_EVENT: hook.event,
      CLAUDE_TOOL_NAME: context.toolName,
      CLAUDE_TOOL_PATH: context.path,
      CLAUDE_TOOL_ARGS: JSON.stringify(context.args || {})
    };
    
    const result = spawn('bash', [hook.path, env], {
      cwd: context.workingDir,
      stdio: ['pipe', 'pipe', 'stderr']
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

### 🔗 钩子链式执行

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
    context:-context: HookContext
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

## 📚 实战案例：安全防护钩子系统

### 需求
创建一个完整的安全防护钩子系统，保护重要文件、防止危险操作。

### 实现

#### 1. 敏感信息检查钩子

```bash
#!/bin/bash
# .claude/hooks/check-secrets.sh

FILES=$(git diff --cached --name-only)

for FILE in $FILES; do
  # 检查API密钥
  if git diff --cached "$FILE" | grep -iEi "(api[_-]?key|secret|password|token)\s*[=:]\s*[\"\']+"; then
    echo "❌ $FILE 包含敏感信息"
    echo "请移除敏感信息后再提交"
    exit 1
  fi
  
  # 检查私钥
  if git diff --cached "$FILE" | grep -EiEi "BEGIN.*PRIVATE"; then
    echo "❌ $FILE 包含私钥"
    echo "请移除私钥后再提交"
    exit 1
  fi
done

echo "✅ 敏感信息检查通过"
exit 0
```

#### 2. 危险命令拦截钩子

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
    echo "⚠️  检测到危险命令：$cmd"
    echo "请确认是否继续"
    exit 1
  fi
done

exit 0
```

#### 3. 配置

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write",
        "hooks": [".claude/hooks/check-secrets.sh"]
      },
      {
        "matcher": "Bash",
        "hooks": [".claude/hooks/check-dangerous.sh"]
      }
    ]
  }
}
```

---

## ✅ 章节总结

### 入门级要点
- ✅ 理解钩子的概念和作用
- ✅ 掌握基本使用方法
- ✅ 了解钩子类型和事件

### 中级要点
- ✅ 掌握钩子配置格式
- 理解钩子脚本编写
- 学会创建自定义钩子
- 掌握匹配器模式

### 专家级要点
- ✅ 深入钩子引擎实现
- 掌握钩子链式执行
- 理解性能优化
- 掌握安全机制实现

### 📊 相关图表

- **钩子执行时序图**：展示事件触发的完整流程
- **Hook链式执行图**：展示多个Hook按顺序执行的机制
- **Hook安全防护流程图**：展示安全检查的完整流程

**详细图表**：[📊 可视化图表集](./VISUAL_GUIDE.md#钩子系统)

---

**下一步：** 学习 [07 - MCP协议](./07-mcp-protocol.md) 🚀
