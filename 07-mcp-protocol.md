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

#### 1️⃣ 工具访问

```markdown
✅ 访问文件系统
✅ 查询数据库
✅ 执行命令行
✅ 调用HTTP API
✅ 连接第三方服务
```

#### 2️⃣ 数据交换

```markdown
✅ 读取文件
✅ 写入文件
✅ 读取API响应
✅ 生成报告
```

#### 3️⃣ 实时监控

```markdown
✅ 监听文件变化
✅ 实时数据查询
✅ 事件通知
```

---

### 🎯 官方MCP服务器

| 服务器 | 功能 | 用途 |
|--------|------|------|
| **github** | GitHub集成 | PR管理、Issue跟踪 |
| **filesystem** | 文件系统 | 文件管理 |
| **database** | 数据库 | 数据查询 |
| **brave-search** | 搜索引擎 | 信息检索 |
| **sequential-thinking** | 思维链 | 复杂推理 |
| **memory** | 记忆系统 | 长期记忆 |

---

### 🌟 实际应用

#### 1. GitHub 集成

```bash
# 安装 GitHub MCP
claude> /mcp add github -- npx -y @modelcontextprotocol/server-github

# 使用GitHub功能
claude> 查看最近的PR
claude> 创建新的PR
claude> 查看Issue #123
```

**效果**：
- 自动连接GitHub API
- 查看和管理PR
- 管理Issue
- 代码审查

#### 2. 数据库查询

```bash
# 连接数据库
claude> /mcp add database -- npx -y @modelcontextprotocol/server-postgres \
    postgresql://user:pass@localhost/mydb

# 查询数据
claude> 查询最近的100个用户

# Claude会自动：
# 1. 连接数据库
# 2. 执行SQL查询
# 3. 格式化结果
# 4. 显示给用户
```

#### 3. 文件系统操作

```bash
# 安装文件系统MCP
claude> /mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem /path/to/allowed

# 使用文件系统功能
claude> 列出指定目录的文件
claude> 搜索特定模式的文件
claude| 批量处理文件
```

---

## 🟡 中级：MCP配置与集成

### 🔧 MCP配置格式

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
      "args": ["-y", "@modelcontextprotocol/server-postgres", "postgres://user:pass@localhost:5432/mydb"]
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allowed"],
      "env": {}
    }
  }
}
```

**配置字段说明**：

| 字段 | 说明 | 示例 |
|------|------|------|
| `command` | 执行命令 | `npx`, `node`, `python` |
| `args` | 命令参数 | `["@modelcontextprotocol/server-github"]` |
| `env` | 环境变量 | `{ "API_KEY": "${API_KEY}" }` |

---

### 🎯 MCP服务器类型

#### 1. stdio服务器

**特点**：
- 通过标准输入输出通信
- 适合本地服务器
- 轻量级

**配置示例**：
```json
{
  "mcpServers": {
    "stdio-server": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-example"]
    }
  }
}
```

#### 2. SSE服务器

**特点**：
- 通过HTTP SSE通信
- 适合远程服务器
- 支持实时更新

**配置示例**：
```json
{
  "mcpServers": {
    "sse-server": {
      "url": "https://example.com/mcp",
      "headers": {
        "Authorization": "Bearer ${TOKEN}"
      }
    }
  }
}
```

#### 3. WebSocket服务器

**特点**：
- 通过WebSocket通信
- 双向实时通信
- 适合实时场景

**配置示例**：
```json
{
  "mcpServers": {
    "websocket-server": {
      "url": "wss://example.com/mcp",
      "headers": {
        "Authorization": "Bearer ${TOKEN}"
      }
    }
  }
}
```

---

## 🔴 专家级：MCP深度剖析

### 🏗️ MCP客户端实现

```typescript
class MCPClient {
  private servers: Map<string, MCPServer>;
  private transports: Map<string, Transport>;
  
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
  
  private async launchServer(config: MCPServerConfig): Promise<MCPServer> {
    let transport: Transport;
    
    if (config.command) {
      // stdio 服务器
      transport = new StdioServerTransport(config.command, config.args, config.env);
    } else if (config.url) {
      // HTTP/SSE 服务器
      transport = new SSEServerTransport(config.url, config.headers);
    }
    
    const server: MCPServer = {
      name: config.name,
      transport,
      tools: []
    };
    
    await transport.connect();
    
    return server;
  }
  
  private async handshake(server: MCPServer): Promise<void> {
    // 发送初始化请求
    const response = await server.transport.send({
      jsonrpc: '2.0',
      id: generateId(),
      method: 'initialize',
      params: {
        capabilities: {
          tools: {},
          resources: {}
        }
      }
    });
    
    // 发送initialized通知
    await server.transport.send({
      jsonrpc: '2.0',
      method: 'notifications/initialized',
      params: {}
    });
  }
}
```

---

### 🔄 工具调用流程

```mermaid
sequenceDiagram
    participant C as Claude Code
    participant Client as MCP Client
    participant Server as MCP Server
    participant Tool as 工具
    
    C->>Client: 调用工具
    Client->>Server: 发送工具调用请求
    Server->>Server: 验证权限
    Server->>Tool: 执行工具
    Tool-->>Server: 返回结果
    Server-->>Client: 返回工具结果
    Client-->>C: 返回最终结果
```

---

### 📊 MCP通信协议

#### 初始化握手

```typescript
// 客户端发送
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "initialize",
  "params": {
    "capabilities": {
      "tools": {},
      "resources": {}
    },
    "clientInfo": {
      "name": "claude-code",
      "version": "1.0.0"
    }
  }
}

// 服务器响应
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "capabilities": {
      "tools": {},
      "resources": {}
    },
    "serverInfo": {
      "name": "my-mcp-server",
      "version": "1.0.0"
    }
  }
}

// 客户端发送初始化完成通知
{
  "jsonrpc": "2.0",
  "method": "notifications/initialized",
  "params": {}
}
```

#### 工具调用

```typescript
// 客户端发送
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "tools/call",
  "params": {
    "name": "get_files",
    "arguments": {
      "path": "/path/to/directory",
      "pattern": "*.ts"
    }
  }
}

// 服务器响应
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "Files:\n- file1.ts\n- file2.ts"
      }
    ]
  }
}
```

---

### ⚡ 性能优化策略

#### 1. 连接池管理

```typescript
class MCPConnectionPool {
  private connections: Map<string, Connection>;
  private maxConnections: number;
  
  constructor(maxConnections: number = 10) {
    this.connections = new Map();
    this.maxConnections = maxConnections;
  }
  
  async getConnection(serverName: string): Promise<Connection> {
    const connection = this.connections.get(serverName);
    
    if (connection && this.isValid(connection)) {
      return connection;
    }
    
    if (this.connections.size >= this.maxConnections) {
      await this.releaseOldestConnection();
    }
    
    const newConnection = await this.createConnection(serverName);
    this.connections.set(serverName, newConnection);
    
    return newConnection;
  }
  
  private isValid(connection: Connection): boolean {
    return Date.now() - connection.lastUsed < 3600000; // 1小时
  }
}
```

#### 2. 批量操作

```typescript
class MCPBatchOperations {
  async batchCall(
    serverName: string,
    operations: BatchOperation[]
  ): Promise<BatchResult[]> {
    const results: BatchResult[] = [];
    
    for (const operation of operations) {
      try {
        const result = await this.callTool(serverName, operation.tool, operation.args);
        results.push({
          success: true,
          operation,
          result
        });
      } catch (error) {
        results.push({
          success: false,
          operation,
          error: error.message
        });
      }
    }
    
    return results;
  }
}
```

---

## 🚨 故障排查

### 常见问题与解决方案

#### 1. MCP服务器连接失败

**症状**：
```
claude> 连接MCP服务器
[连接失败]
```

**可能原因**：
- 服务器未启动
- 网络连接问题
- 认证失败

**解决方案**：
```bash
# 1. 检查服务器状态
claude> 检查MCP服务器状态

# 2. 检查网络连接
ping server.example.com

# 3. 检查认证信息
claude> 检查MCP认证配置
```

#### 2. 工具调用失败

**症状**：
```
claude> 调用MCP工具
[工具调用失败]
```

**可能原因**：
- 工具不存在
- 参数错误
- 权限不足

**解决方案**：
```bash
# 1. 列出可用工具
claude> 列出MCP工具

# 2. 检查工具参数
claude> 检查工具参数

# 3. 检查权限
claude> 检查MCP权限
```

#### 3. 性能问题

**症状**：
```
claude> 调用MCP工具
[响应缓慢]
```

**可能原因**：
- 网络延迟
- 服务器负载高
- 数据量大

**解决方案**：
```bash
# 1. 使用连接池
claude> 启用MCP连接池

# 2. 使用批量操作
claude| 批量调用MCP工具

# 3. 优化数据量
claude| 限制返回数据量
```

---

## 📊 最佳实践清单

### MCP开发

- [ ] 遵循MCP协议规范
- [ ] 实现完整的错误处理
- [ ] 提供清晰的工具描述
- [ ] 支持批量操作
- [ ] 添加性能监控

### MCP使用

- [ ] 合理配置连接池
- [ ] 使用批量操作
- [ ] 监控连接状态
- [ ] 处理错误和重试
- [ ] 优化数据传输

### MCP安全

- [ ] 使用认证机制
- [ ] 限制访问权限
- [ ] 验证输入参数
- [ ] 记录操作日志
- [ ] 定期更新密钥

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
          limit: { 
            type: 'number',
            description: 'Maximum number of orders to return'
          },
          status: {
            type: 'string',
            description: 'Order status filter',
            enum: ['pending', 'completed', 'cancelled']
          }
        }
      }
    },
    {
      name: 'create_order',
      description: 'Create a new order',
      inputSchema: {
        type: 'object',
        properties: {
          customer_id: { 
            type: 'string',
            description: 'Customer ID'
          },
          items: { 
            type: 'array',
            description: 'Order items',
            items: {
              type: 'object',
              properties: {
                product_id: { type: 'string' },
                quantity: { type: 'number' }
              }
            }
          }
        }
      }
    },
    {
      name: 'get_customers',
      description: 'Get customer information',
      inputSchema: {
        type: 'object',
        properties: {
          limit: {
            type: 'number'
          },
          search: {
            type: 'string',
            description: 'Search term'
          }
        }
      }
    }
  ]
}));

// 处理工具调用
server.setRequestHandler('tools/call', async (request) => {
  const { name, arguments: args } = request.params;
  
  switch (name) {
    case 'get_orders':
      return {
        content: [{
          type: 'text',
          text: JSON.stringify(await getOrders(args))
        }]
      };
      
    case 'create_order':
      return {
        content: [{
          type: 'text',
          text: JSON.stringify(await createOrder(args))
        }]
      };
      
    case 'get_customers':
      return {
        content: [{
          type: 'text',
          text: JSON.stringify(await getCustomers(args))
        }]
      };
      
    default:
      throw new Error(`Unknown tool: ${name}`);
  }
});

async function getOrders(args) {
  const limit = args.limit || 10;
  const status = args.status;
  
  // 模拟数据库查询
  const orders = [
    { id: 1, customer_id: 'cust1', items: [{ product_id: 'prod1', quantity: 2 }], status: 'completed' },
    { id: 2, customer_id: 'cust2', items: [{ product_id: 'prod2', quantity: 1 }], status: 'pending' },
    { id: 3, customer_id: 'cust3', items: [{ product_id: 'prod1', quantity: 5 }], status: 'cancelled' }
  ];
  
  return status 
    ? orders.filter(o => o.status === status).slice(0, limit)
    : orders.slice(0, limit);
}

async function createOrder(args) {
  const order = {
    id: Date.now(),
    customer_id: args.customer_id,
    items: args.items,
    status: 'pending',
    created_at: new Date().toISOString()
  };
  
  // 这里应该保存到数据库
  console.log('Created order:', order);
  
  return order;
}

async function getCustomers(args) {
  const limit = args.limit || 10;
  const search = args.search;
  
  // 模拟数据库查询
  const customers = [
    { id: 'cust1', name: 'John Doe', email: 'john@example.com' },
    { id: 'cust2', name: 'Jane Smith', email: 'jane@example.com' },
    { id: 'cust3', name: 'Bob Johnson', email: 'bob@example.com' }
  ];
  
  return search
    ? customers.filter(c => 
        c.name.toLowerCase().includes(search.toLowerCase()) ||
        c.email.toLowerCase().includes(search.toLowerCase())
      ).slice(0, limit)
    : customers.slice(0, limit);
}

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
# 查询订单
claude> 查询最近的10个订单

# Claude会自动：
# 1. 找到MCP服务器
# 2. 调用get_orders工具
# 3. 显示结果

# 创建订单
claude> 创建订单，客户ID是cust1，包含2个prod1

# 查询客户
claude> 搜索名为John的客户
```

---

## ✅ 章节总结

### 入门级要点
- ✅ 理解MCP的概念和价值
- ✅ 掌握基本使用方法
- ✅ 了解官方MCP服务器
- ✅ 学会实际应用

### 中级要点
- ✅ 掌握MCP配置格式
- ✅ 理解不同服务器类型
- ✅ 学会安装和配置MCP服务器
- ✅ 学会使用MCP功能

### 专家级要点
- ✅ 深入MCP客户端实现
- ✅ 掌握工具注册机制
- ✅ 理解自定义服务器开发
- ✅ 掌握通信协议
- ✅ 理解性能优化策略
- ✅ 掌握连接池管理
- ✅ 掌握批量操作

### 📊 相关图表

- **MCP客户端架构图**：展示客户端、服务器、传输的连接机制
- **工具调用流程**：展示工具调用的完整流程
- **自定义服务器架构图**：展示自定义服务器的实现

**详细图表**：[📊 可视化图表集](./VISUAL_GUIDE.md#mcp协议)

---

**下一步：** 学习 [08 - 配置系统](./08-configuration.md) 🚀
