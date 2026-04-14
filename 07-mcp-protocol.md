# 07 - MCP协议集成

## 📋 模块介绍

MCP（Model Context Protocol）是 Claude Code 连接外部工具和服务的标准协议。本章将讲解MCP的配置和使用。

---

## 🟢 入门级：MCP基础认知

### 什么是MCP？

MCP是**模型上下文协议**，让 Claude Code 能够访问外部工具和服务。

**类比理解**：
- 🔌 像USB接口，连接外部设备
- 🌐 像API网关，访问外部服务
- 📦 像插件系统，扩展功能

### MCP能做什么？

```markdown
✅ 访问GitHub - 查看PR、Issue
✅ 查询数据库 - 执行SQL查询
✅ 操作文件系统 - 读写文件
✅ 调用HTTP API - 访问外部服务
✅ 执行系统命令 - 运行CLI工具
```

### 官方MCP服务器

| 服务器 | 功能 | 用途 |
|--------|------|------|
| **github** | GitHub集成 | PR管理、Issue跟踪 |
| **filesystem** | 文件操作 | 高级文件管理 |
| **database** | 数据库查询 | SQL查询执行 |
| **brave-search** | 网络搜索 | 搜索信息 |

---

## 🟡 中级：MCP配置与使用

### MCP配置格式

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "your_token_here"
      }
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allowed"]
    },
    "database": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres", "postgresql://..."]
    }
  }
}
```

### 安装MCP服务器

#### 方法1：使用Claude Code命令

```bash
$ claude
claude> /mcp add github -- npx -y @modelcontextprotocol/server-github
✅ GitHub MCP服务器已添加
```

#### 方法2：手动配置

1. 创建 `~/.claude/settings.json`

2. 添加MCP服务器配置

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    }
  }
}
```

3. 设置环境变量

```bash
export GITHUB_TOKEN="your_token_here"
```

### 使用MCP功能

#### GitHub集成

```bash
$ claude
claude> 查看最近的PR

📋 使用GitHub MCP...

最近的PR：
1. #123 - 添加新功能
2. #122 - 修复bug
3. #121 - 性能优化
```

#### 数据库查询

```bash
claude> 查询最近的100个订单

📊 使用数据库MCP...

查询结果：
- 订单总数：100
- 总金额：$50,000
- 平均金额：$500
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

  private async launchServer(
    config: MCPServerConfig
  ): Promise<MCPServer> {
    // 启动服务器进程
    const process = spawn(config.command, config.args, {
      env: {
        ...process.env,
        ...config.env
      }
    });

    // 创建通信通道
    const transport = new StdioTransport(process.stdin, process.stdout);

    return {
      name: config.name,
      process,
      transport,
      capabilities: {}
    };
  }

  async callTool(
    serverName: string,
    toolName: string,
    args: any
  ): Promise<any> {
    const server = this.servers.get(serverName);

    if (!server) {
      throw new Error('Server not connected: ' + serverName);
    }

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
}
```

### MCP工具注册

```typescript
class MCPToolRegistry {
  private tools: Map<string, MCPTool>;

  register(server: string, tools: MCPTool[]): void {
    for (const tool of tools) {
      const key = `${server}:${tool.name}`;
      this.tools.set(key, {
        ...tool,
        server
      });
    }
  }

  find(toolName: string): MCPTool | null {
    // 精确匹配
    if (this.tools.has(toolName)) {
      return this.tools.get(toolName)!;
    }

    // 模糊匹配
    for (const [key, tool] of this.tools.entries()) {
      if (key.endsWith(toolName)) {
        return tool;
      }
    }

    return null;
  }
}
```

---

## 📚 实战案例：集成自定义MCP服务

### 需求
- 🔗 连接到内部API服务
- 📊 获取业务数据
- 🎯 自动化业务流程

### 实现

#### 1. 创建MCP服务器

```javascript
// my-mcp-server/index.js
const { Server } = require('@modelcontextprotocol/sdk/server');
const { StdioServerTransport } = require('@modelcontextprotocol/sdk/server/stdio');

const server = new Server({
  name: 'my-business-api',
  version: '1.0.0'
}, {
  capabilities: {
    tools: {}
  }
});

// 注册工具
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
          items: { type: 'array' }
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
          text: JSON.stringify(await getOrders(args.limit))
        }]
      };

    case 'create_order':
      return {
        content: [{
          type: 'text',
          text: JSON.stringify(await createOrder(args))
        }]
      };
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
$ claude
claude> 查询最近的10个订单

📊 使用business-api MCP...

查询结果：
[
  {"id": "001", "amount": 100},
  {"id": "002", "amount": 200},
  ...
]
```

---

## ✅ 章节总结

### 入门级要点
- ✅ 理解MCP的概念
- ✅ 掌握基本使用方法
- ✅ 了解官方MCP服务器

### 中级要点
- ✅ 掌握MCP配置格式
- ✅ 理解服务器安装
- ✅ 学会使用MCP功能

### 专家级要点
- ✅ 深入MCP客户端实现
- ✅ 掌握工具注册
- ✅ 理解自定义服务器开发

---

**下一步：** 学习 [08 - 配置系统](./08-configuration.md) 🚀
