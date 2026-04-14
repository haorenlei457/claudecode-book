# 04 - 代理系统（Agents）

## 📋 模块介绍

代理系统是 Claude Code 的"专家团队"概念，通过专业化AI助手完成特定任务。每个代理都有专长领域。本章将通过大量实例和图表，深入讲解代理的设计、协作和管理。

---

## 🟢 入门级：代理基础认知

### 🤔 什么是代理？

#### 简单理解

**代理（Agent）就像你的"专业团队"**，每个代理都是一个"专家"，负责特定领域的工作。

**类比理解**：

```
传统方式（通用Claude）：
你：帮我做A、B、C三件事
Claude：我来做A、B、C

使用代理（专业化）：
你：帮我做开发任务
Claude：@backend-developer 负责后端
Claude：@frontend-developer 负责前端
Claude：@test-engineer 写测试
Claude：@code-reviewer 审查代码
```

**核心优势：**
- 🎯 **专业化** - 每个代理专注于特定领域
- 🧠 **深度理解** - 针对性训练和配置
- 🔁 **质量高** - 专业的问题检查和建议
- 🤝 **协作** - 多个代理分工协作

---

### 🎯 代理 vs 其他概念

#### 代理 vs 命令

| 特性 | 代理 | 命令 |
|------|------|------|
| 触发方式 | 自动委派 | 手动输入 |
| 智能程度 | 高（AI决策） | 低（预设流程） |
| 复杂度 | 高（处理复杂任务） | 低（简单操作） |
| 实时性 | 实时反馈 | 一次性执行 |
| 示例 | @code-reviewer | /commit |

**实际对比：**

```bash
# 使用命令
claude> /review
结果：直接执行预定义流程

# 使用代理
claude> 审查这个文件的质量
Claude: 自动调用 @code-reviewer 代理进行深度分析
结果：更专业的审查报告
```

#### 代理 vs 技能

| 特性 | 代理 | 技能 |
|------|------|------|
| 触发方式 | 自动委派 | 自动触发 |
| 智能程度 | 高（理解+决策） | 低（预设规则） |
| 复杂度 | 中高（处理复杂场景） | 低（单一功能） |
| 协作能力 | 能委派其他代理 | 不能 |
| 示例 | @code-reviewer | code-review |

**实际对比：**

```bash
# 使用技能
claude> 帮我格式化代码
Claude: 自动触发 code-formatter 技能

# 使用代理
claude> 审查这个文件的代码
Claude: 自动委派 @code-reviewer 代理，可以深入分析
```

---

### 🌟 官方代理示例

#### 1️⃣ code-reviewer（代码审查员）

**职责**：审查代码质量、安全性和最佳实践

**能力**：
- 代码风格检查
- 性能优化建议
- 安全漏洞检测
- 最佳实践建议
- 正面评价

**使用方式**：
```bash
claude> 审查 src/main.py

# Claude自动选择code-reviewer代理
```

#### 2️⃣ test-engineer（测试工程师）

**职责**：编写测试、提高覆盖率

**能力**：
- 单元测试
- 集成测试
- 测试覆盖率分析
- 性能测试
- Mock数据生成

**使用方式**：
```bash
claude> 为这个函数编写测试
# Claude自动选择test-engineer代理
```

#### 3️⃣ documentation-writer（文档编写者）

**职责**：生成技术文档

**能力**：
- API文档
- 代码注释
- README编写
- Wiki文档

**使用方式**：
```bash
claude> 生成API文档
# Claude自动使用documentation-writer技能
```

#### 4️⃣ api-designer（API设计师）

**职责**：设计API接口

**能力**：
- RESTful设计
- OpenAPI规范
- 接口文档生成

**使用方式**：
```bash
claude> 设计用户认证API
# Claude自动使用api-designer代理
```

#### 5️⃣ security-auditor（安全审计员）

**职责**：安全检查和风险评估

**能力**：
- SQL注入检测
- XSS漏洞
- 权限验证
- 敏感信息检查

**使用方式**：
```bash
claude> 检查代码安全性
# Claude自动使用security-auditor代理
```

---

### 🤝 代理协作模式

#### 串行协作

```mermaid
graph LR
    A[用户任务] --> B[@code-architect: 设计架构]
    B --> C[@implementation-agent: 实现代码]
    C --> D[@test-engineer: 编写测试]
    D --> E[@code-reviewer: 审查代码]
    E --> F[返回结果]
```

**示例**：
```bash
claude> 开发用户认证功能

# 串行执行：
1. 设计架构（架构师）
2. 实现代码（实现者）
3. 编写测试（测试员）
4. 审查代码（审查员）
```

#### 并行协作

```mermaid
graph TD
    A[用户任务] --> B[@code-reviewer: 代码质量]
    A --> C[@security-auditor: 安全检查]
    A --> D[@performance-analyst: 性能分析]
    A --> E[@documentation-writer: 文档编写]
    
    B --> F{质量检查}
    C --> G{安全检查}
    D --> H{性能分析}
    E --> I{文档编写}
    
    F --> J[汇总质量检查结果]
    G --> J
    H --> J
    I --> J
    H --> J
    I --> J
    
    J --> K[返回完整报告]
```

**示例**：
```bash
claude> 审查代码质量和安全性

# 并行执行：
1. @code-reviewer 检查代码质量
2. @security-auditor 检查安全性
3. @performance-analyst 分析性能
4. @documentation-writer 生成文档

# 三个代理并行执行，最后汇总结果
```

#### 主从协作

```mermaid
graph TD
    A[用户任务] --> B[@orchestrator: 协调器]
    B --> C[@frontend-dev: 前端开发]
    B --> D[@backend-dev: 后端开发]
    B --> E[@test-engineer: 测试工程师]
    B --> F[@reviewer: 审查]
    
    C --> G[前端实现]
    D --> H[后端实现]
    E --> I[测试代码]
    F --> J[代码审查]
    
    G --> K[检查前端]
    H --> K
    I --> K
    J --> K
    J --> K
    
    K --> L[返回结果]
    B --> L
```

**示例**：
```bash
claude> 完成新功能

# 协调器协调：
1. @frontend-dev 开发前端
2. @backend-dev 开发后端
3. @test-engineer 编写测试
4. @reviewer 审查代码
5. 协调器汇总结果
```

---

## 🟡 中级：代理开发与协作

### 📝 代理定义格式

```markdown
---
id: "代理ID"
name: "代理名称"
role: "代理角色"
description: "代理描述"
version: "版本号"
permissions:
  - "file:read"
  - "git:diff"
  - "file:write"
capabilities:
  - "能力1"
  - "能力2"
  - "能力3"
tools:
  - "file:read"
  - "file:write"
  - "git:diff"
  - "git:log"
---

你是[详细描述]

## 职责
- [职责1]
- [职责2]
- [职责3]

## 审查标准
- [标准1]
- [标准2]
- [标准3]

## 输出格式
[输出格式]
```

## 注意事项
- [注意1]
- [注意2]
```

---

**实际示例**：

```markdown
---
id: "code-reviewer"
name: "Code Reviewer"
role: "Quality Assurance"
description: "Specialized agent for code review"
version: "1.0.0"
permissions:
  - "file:read"
  - "git:diff"
  - "git:log"
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
- 文档注释

### 安全性
- SQL注入风险
- XSS漏洞
- 权限验证
- 敏感信息

### 最佳实践
- 设计模式
- 代码复用
- 注释文档
- 测试覆盖
```

---

### 实战应用

#### 使用场景
用户：
> 审查 src/main.py

Claude 自动：
1. 读取文件内容
2. 分析代码质量
3. 检查安全性
4. 生成审查报告
5. 提供改进建议
```

---

## ✅ 章节总结

### 入门级要点
- ✅ 理解代理的概念和优势
- ✅ 掌握代理的基本使用方法
- ✅ 了解官方代理示例

### 中级要点
- ✅ 掌握代理定义格式
- ✅ 理解代理协作模式
- ✅ 学会创建自定义代理
- ✅ 掌握能力定义方法

### 专家级要点
- ✅ 深入代理注册系统
- 掌握代理委派机制
- 理解性能监控
- 掌握安全策略
- 理解协作编排

### 📊 相关图表
- **代理协作流程图**：串行、并行、主从三种模式
- **代理委派决策图**：选择合适代理的过程
- **代理评分算法图**：代理评分和选择算法

**详细图表**：[📊 可视化图表集](./VISUAL_GUIDE.md#代理系统)

---

**下一步：** 学习 [05 - 技能系统](./05-skill-system.md) 🚀
