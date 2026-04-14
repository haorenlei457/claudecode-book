# 07 - MCP协议集成

## 📋 模块介绍

MCP（Model Context Protocol）是 Claude Code 连接外部工具和服务的标准协议。本章将讲解MCP的概念、配置方法和实际应用。

---

## 🟢 入门级：MCP基础认知

### 🤔 什么是MCP？

#### 简单理解

**MCP（Model Context Protocol）就像是 Claude Code 的"通用接口标准"**，让Claude Code能够与外部工具和服务对话。

**类比理解**：

```
传统方式：
Claude Code 内置能力 → 可用的工具（文件、Git、终端）

使用MCP后：
Claude Code → MCP协议 → 任何外部工具或服务
```

**核心价值**：
- 🌐 可扩展：支持无限扩展
- 🔌 标准化：统一的使用方式
- 🔧 灵活性：自动发现和配置
- 📊 可组合：多个MCP服务器协同工作

---

### 🌐 MCP能做什么？

```markdown
---
name: "MCP能力展示"
---

## 1. 工具访问
- 访问文件系统
- 查询数据库
- 执行命令行

## 2. 数据交换
- 读取文件
- 写入文件
- 读取API响应

## 3. 实时监控
- 监听文件变化
- 实时数据查询
```

---

### 🎯 官方MCP服务器

| 服务器 | 功能 | 用途 |
|--------|------|------|
| **github** | GitHub集成 | PR管理、Issue跟踪 |
| **filesystem** | 文件系统 | 文件管理 |
| **database** | 数据库 | 数据查询 |
| **brave-search** | 搜索引擎 | 信息检索 |

---

### 实际应用

#### 1. GitHub 集成

```bash
# 安装 GitHub MCP
claude> /mcp add github -- npx -y @modelcontextprotocol/server-github

# 使用GitHub功能
claude> 查看最近的PR
claude> 创建新的PR
claude> 查看Issue #123
```

#### 2. 数据库查询

```bash
# 连接数据库
claude> /mcp add database -- npx -y @modelcontextprotocol/server-postgres \
    postgresql://user:pass@localhost/mydb

# 查询数据
claude> 查询最近的100个用户
```

---

## 🟡 中级：MCP配置与集成

### MCP配置格式

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "database": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "args": ["-y", "postgres://user:pass@localhost:5432/mydb"]
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allowed"],
      "env": {}
    }
  }
}
```

---

## 🔴 专家级：MCP深度剖析

### MCP客户端实现

```typescript
class MCPClient {
  private servers: Map<string, MCPServer>;
  
  async connect(serverName: string): Promise<void> {
    const config = this.getServerConfig(serverName);
    const server = await this.launchServer(config);
    this.servers.set(serverName, server);
    
    // 初始化握手
    await this.handshake(server);
  }
  
  async callTool(
    serverName: string,
    toolName: string,
    args: any
  ): Promise<any> {
    const server = this.servers.get(serverName);
    
    // 发送工具调用请求
    const response = await server.transport.send({
      jsonrpc: '2.0',
      id: generateId(),
      method: 'tools/call',
      params: {
        name: toolName,
        arguments: args
      }
    });
    
    return response.result;
  }
  
  async listTools(
    serverName: string
  ): Promise<Tool[]> {
    const server = this.servers.get(serverName);
    
    // 列出工具
    const response = await server.transport.send({
      jsonrpc: '2.0',
      id: generateId(),
      method: 'tools/list',
      params: {}
    });
    
    return response.result.tools || [];
  }
}
```

---

## 📚 实战案例：自定义MCP服务器

### 需求
创建一个业务数据查询的MCP服务器。

### 实现

#### 1. 创建服务器

```javascript
// my-mcp-server/index.js
const { Server } = require('@modelcontextprotocol/sdk/server');

const server = new Server({
  name: 'my-business-api',
  version: '1.0.0'
});

server.setRequestHandler('tools/list', async () => ({
  tools: [
    {
      name: 'get_orders',
      description: 'Get recent orders',
      inputSchema: {
        type: 'object',
        properties: {
          limit: { type: 'number' }
        }
      }
    },
    {
      name: 'create_order',
      description: 'Create a new order',
      inputSchema: {
        type: 'object',
        properties: {
          customer_id: { type: 'string' },
          items: { type: 'array', items: { type: 'object' } }
        }
      }
    }
  ]
}));

// 处理工具调用
server.setRequestHandler('tools/call', async (request) => {
  const { name, arguments } = request.params;
  
  switch (name) {
    case 'get_orders':
      return {
        content: [{
          type: 'text',
          text: JSON.stringify(await getOrders(arguments.limit))
        }]
      };
      
    case 'create_order':
      return {
        content: [{
          type: 'text',
          text: JSON.stringify(await createOrder(arguments))
        }]
      };
      
    default:
      throw new Error(`Unknown tool: ${name}`);
  }
});

async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
}

main();
```

#### 2. 配置MCP服务器

```json
{
  "mcpServers": {
    "business-api": {
      "command": "node",
      "args": ["my-mcp-server/index.js"]
    }
  }
}
```

#### 3. 使用

```bash
claude> 查询最近的10个订单

# Claude会自动：
# 1. 找动服务器
# 2. 调用get_orders工具
# 3. 显示结果
```

---

## ✅ 章节总结

### 入门级要点
- ✅ 理解MCP的概念和价值
- ✅ 掌握基本使用方法
- 了解官方MCP服务器

### 中级要点
- ✅ 掌握MCP配置格式
- 学会安装和配置MCP服务器
- 学会使用MCP功能

### 专家级要点
- ✅ 深入MCP客户端实现
- 掌握工具注册机制
- 理解自定义服务器开发
- 掌握安全机制

### 📊 相关图表

- **MCP客户端架构图**：展示客户端、服务器、传输的连接机制
- **工具调用流程**：展示工具调用的完整流程
- **自定义服务器架构图**：展示自定义服务器的实现

---

**下一步：** 学习 [08 - 配置系统](./08-configuration.md) 🚀
