# 🎯 Claude Code 实战项目集

## 📋 项目列表

本集合包含5个完整的实战项目，从入门到高级，帮助你掌握 Claude Code 的实际应用。

---

## 项目1：个人代码助手 ⭐

### 目标
创建一个个性化的代码助手插件，提供常用代码片段和最佳实践建议。

### 难度
⭐ (入门)

### 预计时间
30分钟

### 步骤

#### 1. 创建插件结构

```bash
mkdir -p ~/.claude/plugins/code-assistant/{commands,skills}
```

#### 2. 创建命令

```markdown
# ~/.claude/plugins/code-assistant/commands/snippets.md
---
name: "snippet"
description: "Generate code snippets"
---

根据需求生成代码片段。

## 可用片段

### Python 函数模板
```python
def {{function_name}}({{parameters}}):
    """
    {{description}}
    
    Args:
        {{args}}
    
    Returns:
        {{return_type}}
    """
    {{implementation}}
    
    return {{return_value}}
```

### JavaScript 异步函数
```javascript
async function {{function_name}}({{parameters}}) {
  try {
    {{implementation}}
    return {{return_value}};
  } catch (error) {
    console.error('Error:', error);
    throw error;
  }
}
```
```

#### 3. 创建技能

```markdown
# ~/.claude/plugins/code-assistant/skills/best-practices/SKILL.md
---
name: "best-practices"
description: "Provide coding best practices"
triggers:
  - "best practice"
  - "代码规范"
  - "最佳实践"
---

## Python 最佳实践

### 命名规范
- 函数：snake_case
- 类：PascalCase
- 常量：UPPER_SNAKE_CASE

### 错误处理
```python
try:
    result = risky_operation()
except SpecificException as e:
    logger.error(f"Operation failed: {e}")
    raise CustomException("Message") from e
```

### 类型注解
```python
from typing import List, Optional

def process_items(items: List[str]) -> Optional[dict]:
    ...
```
```

#### 4. 测试

```bash
$ claude
claude> /snippet python function

# 生成Python函数模板
```

---

## 项目2：智能 Git 工作流 ⭐⭐

### 目标
创建一个智能 Git 工作流系统，自动处理提交、审查和合并。

### 难度
⭐⭐ (中级)

### 预计时间
45分钟

### 步骤

#### 1. 创建钩子

```bash
mkdir -p .claude/hooks
```

```bash
# .claude/hooks/pre-commit.sh
#!/bin/bash

echo "🧪 运行测试..."
npm test

if [ $? -ne 0 ]; then
    echo "❌ 测试失败，阻止提交"
    exit 1
fi

echo "🔍 检查代码格式..."
npm run lint

if [ $? -ne 0 ]; then
    echo "❌ 代码格式检查失败"
    echo "运行 'npm run lint:fix' 修复"
    exit 1
fi

echo "✅ 预提交检查通过"
exit 0
```

#### 2. 创建命令

```markdown
# .claude/commands/smart-commit.md
---
name: "smart-commit"
description: "Smart commit with checks"
---

智能提交流程：

1. **检查变更**
   {{git:status}}

2. **运行检查**
   - 代码格式检查
   - 静态分析
   - 单元测试

3. **生成提交消息**
   分析变更内容，生成符合规范的提交消息

4. **执行提交**
   {{git:add .}}
   {{git:commit -m "{{generated_message}}"}}
```

#### 3. 配置

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

---

## 项目3：代码审查自动化 ⭐⭐⭐

### 目标
创建自动化的代码审查系统，检查代码质量、安全性和最佳实践。

### 难度
⭐⭐⭐ (高级)

### 预计时间
60分钟

### 步骤

#### 1. 创建审查代理

```markdown
# .claude/agents/code-reviewer.md
---
id: "auto-reviewer"
name: "Auto Code Reviewer"
role: "Automated Code Review"
description: "Automated code quality and security review"
permissions:
  - "file:read"
  - "git:diff"
---

## 审查清单

### 代码质量
- [ ] 遵循项目编码规范
- [ ] 函数长度 < 50行
- [ ] 圈复杂度 < 10
- [ ] 有适当的注释

### 安全性
- [ ] 无SQL注入风险
- [ ] 无XSS漏洞
- [ ] 敏感信息未硬编码
- [ ] 输入已验证

### 性能
- [ ] 无N+1查询
- [ ] 无内存泄漏
- [ ] 合理使用缓存

## 输出格式
```markdown
## 审查报告

### 问题汇总
| 严重程度 | 数量 |
|----------|------|
| 🔴 严重 | {{critical}} |
| 🟡 警告 | {{warning}} |
| 💡 建议 | {{suggestion}} |

### 详细问题
{{issues}}

### 正面评价
{{positives}}
```
```

#### 2. 创建审查命令

```markdown
# .claude/commands/review-code.md
---
name: "review-code"
description: "Automated code review"
---

启动自动代码审查...

@auto-reviewer 审查当前变更

## 审查范围
- 代码质量
- 安全性
- 性能
- 可维护性

## 审查结果
{{review_result}}
```

---

## 项目4：API 文档生成器 ⭐⭐⭐

### 目标
自动从代码生成 API 文档，支持 OpenAPI 规范。

### 难度
⭐⭐⭐ (高级)

### 预计时间
75分钟

### 实现

```typescript
// scripts/generate-api-docs.ts
import * as fs from 'fs';
import * as path from 'path';
import { parse } from '@babel/parser';
import traverse from '@babel/traverse';

interface APIEndpoint {
  method: string;
  path: string;
  description: string;
  parameters: Parameter[];
  responses: Response[];
}

function generateAPIDocs(sourceDir: string): void {
  const endpoints: APIEndpoint[] = [];

  // 扫描路由文件
  const routeFiles = findFiles(sourceDir, ['.ts', '.js']);

  for (const file of routeFiles) {
    const content = fs.readFileSync(file, 'utf-8');
    const ast = parse(content, {
      sourceType: 'module',
      plugins: ['typescript', 'decorators-legacy']
    });

    traverse(ast, {
      CallExpression(nodePath) {
        const { callee } = nodePath.node;

        // 检测路由定义
        if (callee.type === 'MemberExpression') {
          const method = callee.property.name;
          if (['get', 'post', 'put', 'delete'].includes(method)) {
            const endpoint = extractEndpoint(nodePath.node);
            endpoints.push(endpoint);
          }
        }
      }
    });
  }

  // 生成 OpenAPI 规范
  const openApiSpec = generateOpenAPISpec(endpoints);

  // 写入文件
  fs.writeFileSync(
    'openapi.json',
    JSON.stringify(openApiSpec, null, 2)
  );
}

function generateOpenAPISpec(endpoints: APIEndpoint[]): object {
  return {
    openapi: '3.0.0',
    info: {
      title: 'API Documentation',
      version: '1.0.0'
    },
    paths: endpoints.reduce((acc, endpoint) => {
      acc[endpoint.path] = {
        [endpoint.method]: {
          summary: endpoint.description,
          parameters: endpoint.parameters,
          responses: endpoint.responses.reduce((r, res) => {
            r[res.status] = { description: res.description };
            return r;
          }, {})
        }
      };
      return acc;
    }, {})
  };
}
```

---

## 项目5：完整 CI/CD 流水线 ⭐⭐⭐⭐

### 目标
构建完整的 CI/CD 流水线，包括测试、构建、部署和监控。

### 难度
⭐⭐⭐⭐ (专家)

### 预计时间
90分钟

### 架构

```
代码提交
  ↓
预提交检查 (钩子)
  ↓
自动化测试
  ↓
代码审查 (代理)
  ↓
构建
  ↓
部署到 staging
  ↓
集成测试
  ↓
部署到 production
  ↓
监控和告警
```

### 实现

#### 1. 预提交钩子

```bash
# .claude/hooks/pre-push.sh
#!/bin/bash

echo "🚀 预推送检查..."

# 运行测试
echo "🧪 运行测试..."
npm test || exit 1

# 检查覆盖率
echo "📊 检查覆盖率..."
npm run test:coverage || exit 1

# 安全检查
echo "🔒 安全检查..."
npm audit || exit 1

echo "✅ 所有检查通过"
exit 0
```

#### 2. 部署代理

```markdown
# .claude/agents/deployment-agent.md
---
id: "deployment-agent"
name: "Deployment Agent"
role: "Automated Deployment"
description: "Handle deployment to various environments"
permissions:
  - "bash:run"
  - "file:read"
---

## 部署流程

### Staging 部署
1. 构建应用
2. 运行数据库迁移
3. 部署到 staging 服务器
4. 运行健康检查
5. 发送通知

### Production 部署
1. 检查 staging 状态
2. 创建备份
3. 执行部署
4. 验证部署
5. 监控指标

## 回滚策略
- 保留最近 3 个版本
- 自动回滚条件：
  - 错误率 > 5%
  - 响应时间 > 2s
  - 健康检查失败
```

#### 3. 监控集成

```typescript
// scripts/monitor-deployment.ts
import axios from 'axios';

interface HealthCheck {
  url: string;
  expectedStatus: number;
  timeout: number;
}

async function monitorDeployment(healthChecks: HealthCheck[]): Promise<boolean> {
  for (const check of healthChecks) {
    try {
      const response = await axios.get(check.url, {
        timeout: check.timeout
      });

      if (response.status !== check.expectedStatus) {
        console.error(`❌ Health check failed: ${check.url}`);
        return false;
      }
    } catch (error) {
      console.error(`❌ Health check error: ${check.url}`);
      return false;
    }
  }

  console.log('✅ All health checks passed');
  return true;
}
```

---

## 🎓 学习建议

### 学习顺序

1. **项目1** - 熟悉基本概念
2. **项目2** - 掌握Git工作流
3. **项目3** - 深入代理系统
4. **项目4** - 理解代码分析
5. **项目5** - 综合运用所有知识

### 实践建议

- 每个项目完成后，尝试修改和扩展
- 记录遇到的问题和解决方案
- 分享自己的实现到社区

---

祝你实战愉快！🚀
