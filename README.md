# Make.com MCP Server 研究报告

## 概述

本研究报告深入探讨了 Make.com 如何通过 Model Context Protocol (MCP) 提供服务器功能，使 Claude 等 AI 助手能够与 Make.com 的自动化工作流程进行交互，特别关注如何通过 Make.com 使用 GitHub MCP server。

## 什么是 Model Context Protocol (MCP)

### MCP 简介

Model Context Protocol (MCP) 是由 Anthropic 开发的开放标准，旨在为 AI 模型和外部工具、数据源之间建立安全的双向连接。MCP 的核心优势包括：

- **标准化接口**：统一的协议消除了每个 AI 模型和工具之间需要自定义集成的需求
- **安全性**：数据保持在您的基础设施内，同时与 AI 进行交互
- **可扩展性**：支持多种传输方式，如 stdio、WebSockets、HTTP SSE 和 UNIX sockets
- **灵活性**：允许在不同 AI 模型和供应商之间轻松切换

### MCP 架构组件

1. **MCP 主机 (Hosts)**：如 Claude Desktop、VS Code 等应用程序
2. **MCP 客户端 (Clients)**：主机内的连接器，维护与 MCP 服务器的 1:1 有状态会话
3. **MCP 服务器 (Servers)**：提供对外部工具和数据源的访问
4. **工具 (Tools)**：可被 LLM 调用的函数
5. **资源 (Resources)**：类似文件的数据，可被客户端读取
6. **提示 (Prompts)**：预写的模板和工作流程

## Make.com MCP Server 详解

### Make.com MCP Server 的优势

Make.com 的 MCP server 带来了革命性的改进：

1. **云端基础设施**：与大多数需要本地设置的 MCP 服务器不同，Make.com 提供云端托管的解决方案
2. **零代码配置**：只需生成 token 并粘贴 URL，无需 Node.js、Python 或 Docker 知识
3. **实时发现和执行**：通过云端基础设施和独特的 Make MCP Token 保障安全
4. **标准化 API 交互**：简化了与数千个 API 的通信

### Make.com MCP Server 功能

- 检测用户账户中所有设置为"按需"运行的 Make 场景
- 为每个场景返回有意义的输入参数描述
- 支持场景输入和输出的配置
- 提供结构化的数据交换格式

## 设置 Make.com MCP Server

### 前置条件

1. Make.com 账户（所有订阅计划都支持，包括免费计划）
2. 已创建的 Make 场景，设置为"激活"和"按需"模式
3. 支持 MCP 的 AI 工具（如 Claude Desktop、Cursor 等）

### 获取 MCP Token 步骤

1. 登录 Make.com 账户
2. 导航到用户配置文件中的"API / MCP 访问"部分
3. 点击"添加 token"
4. 选择"MCP Token"作为类型
5. 为您的 MCP Token 选择标签
6. 点击"添加"
7. 复制生成的唯一 URL

### Claude Desktop 配置

在 Claude Desktop 的配置文件中添加以下配置：

```json
{
  "mcpServers": {
    "make": {
      "command": "npx",
      "args": ["-y", "@makehq/mcp-server"],
      "env": {
        "MAKE_API_KEY": "<your-api-key>",
        "MAKE_ZONE": "<your-zone>",
        "MAKE_TEAM": "<your-team-id>"
      }
    }
  }
}
```

## GitHub MCP Server 集成

### GitHub MCP Server 概述

GitHub 的官方 MCP Server 提供了与 GitHub API 的标准化接口，支持多种工具集：

- **用户工具**：获取认证用户信息
- **仓库工具**：仓库管理和内容访问
- **问题工具**：创建和管理 GitHub issues
- **Pull Request 工具**：PR 管理和代码审查
- **代码安全工具**：安全扫描和漏洞管理

### GitHub MCP Server 设置

#### 前置条件

1. Docker 环境
2. GitHub Personal Access Token (PAT)

#### 配置步骤

1. **创建 GitHub Personal Access Token**
   - 访问 GitHub Settings > Developer settings > Personal access tokens
   - 生成新的 token 并设置适当的权限

2. **Docker 配置**
   ```bash
   docker run -i --rm \
     -e GITHUB_PERSONAL_ACCESS_TOKEN=<your-token> \
     ghcr.io/github/github-mcp-server
   ```

3. **Claude Desktop 配置**
   ```json
   {
     "mcpServers": {
       "github": {
         "command": "docker",
         "args": [
           "run", "-i", "--rm",
           "-e", "GITHUB_PERSONAL_ACCESS_TOKEN",
           "ghcr.io/github/github-mcp-server"
         ],
         "env": {
           "GITHUB_PERSONAL_ACCESS_TOKEN": "<YOUR_TOKEN>"
         }
       }
     }
   }
   ```

4. **VS Code 配置**
   ```json
   {
     "mcp": {
       "inputs": [
         {
           "type": "promptString",
           "id": "github_token",
           "description": "GitHub Personal Access Token",
           "password": true
         }
       ],
       "servers": {
         "github": {
           "command": "docker",
           "args": [
             "run", "-i", "--rm",
             "-e", "GITHUB_PERSONAL_ACCESS_TOKEN",
             "ghcr.io/github/github-mcp-server"
           ],
           "env": {
             "GITHUB_PERSONAL_ACCESS_TOKEN": "${input:github_token}"
           }
         }
       }
     }
   }
   ```

### 工具集管理

GitHub MCP Server 支持通过 `--toolsets` 标志启用或禁用特定功能组：

```bash
docker run -i --rm \
  -e GITHUB_PERSONAL_ACCESS_TOKEN=<your-token> \
  -e GITHUB_TOOLSETS="repos,issues,pull_requests,code_security" \
  ghcr.io/github/github-mcp-server
```

可用的工具集包括：
- `repos`：仓库操作
- `issues`：问题管理
- `pull_requests`：Pull Request 管理
- `code_security`：代码安全扫描
- `experiments`：实验性功能

## 通过 Make.com 使用 GitHub MCP Server

### 集成策略

1. **Make.com 作为中介**
   - 创建 Make 场景来调用 GitHub API
   - 通过 Make.com MCP Server 暴露这些功能
   - 提供额外的数据处理和转换能力

2. **混合方法**
   - 同时配置 Make.com MCP Server 和 GitHub MCP Server
   - 根据需要选择合适的工具
   - 利用 Make.com 的自动化能力增强 GitHub 操作

### 实施步骤

1. **在 Make.com 中创建 GitHub 集成场景**
   ```
   - 场景 1：GitHub Issue 创建和管理
   - 场景 2：Pull Request 审查工作流
   - 场景 3：仓库监控和通知
   - 场景 4：自动化代码质量检查
   ```

2. **配置场景参数**
   - 输入：仓库名称、issue 标题、描述等
   - 输出：创建的 issue 链接、状态更新等

3. **集成到 MCP 配置**
   ```json
   {
     "mcpServers": {
       "make": {
         "command": "npx",
         "args": ["-y", "@makehq/mcp-server"],
         "env": {
           "MAKE_API_KEY": "<your-api-key>",
           "MAKE_ZONE": "<your-zone>",
           "MAKE_TEAM": "<your-team-id>"
         }
       },
       "github": {
         "command": "docker",
         "args": [
           "run", "-i", "--rm",
           "-e", "GITHUB_PERSONAL_ACCESS_TOKEN",
           "ghcr.io/github/github-mcp-server"
         ],
         "env": {
           "GITHUB_PERSONAL_ACCESS_TOKEN": "<YOUR_TOKEN>"
         }
       }
     }
   }
   ```

## 使用案例

### 1. 自动化项目管理
- **问题跟踪**：通过 Claude 创建和管理 GitHub issues
- **PR 审查**：自动化代码审查流程
- **项目监控**：实时监控仓库状态和活动

### 2. 智能客户反馈处理
- **反馈收集**：通过 Make.com 表单收集用户反馈
- **Issue 创建**：自动将反馈转换为 GitHub issues
- **优先级分配**：基于反馈内容自动分配优先级

### 3. 开发工作流程优化
- **代码质量监控**：自动检查代码质量并创建相应的 issues
- **团队协作**：通过 Claude 协调团队任务和分配
- **文档生成**：自动生成和更新项目文档

## 安全考虑

### 1. Token 安全
- 使用环境变量存储敏感信息
- 定期轮换 API tokens
- 遵循最小权限原则

### 2. 权限管理
- 仔细配置 GitHub PAT 权限
- 定期审查 Make.com 场景权限
- 监控 API 使用情况

### 3. 数据保护
- 确保敏感数据不被记录
- 使用 HTTPS 进行所有通信
- 遵循组织的数据保护政策

## 最佳实践

### 1. 场景设计
- 保持场景简单和专注
- 提供清晰的输入/输出描述
- 实施错误处理和重试逻辑

### 2. 监控和调试
- 启用 Make.com 场景日志
- 监控 API 使用配额
- 设置适当的告警机制

### 3. 文档和维护
- 记录所有场景的用途和配置
- 定期更新和测试集成
- 保持配置文件的版本控制

## 故障排除

### 常见问题

1. **连接问题**
   - 检查 MCP token 有效性
   - 验证网络连接
   - 确认服务器状态

2. **权限错误**
   - 验证 GitHub PAT 权限
   - 检查 Make.com 场景权限
   - 确认用户访问权限

3. **配置错误**
   - 验证 JSON 配置格式
   - 检查环境变量设置
   - 确认文件路径正确

### 调试技巧

1. **日志分析**
   ```bash
   # Claude Desktop 日志
   tail -n 20 -f ~/Library/Logs/Claude/mcp*.log
   
   # Docker 容器日志
   docker logs <container-id>
   ```

2. **MCP Inspector**
   - 使用 MCP Inspector 测试服务器连接
   - 验证工具可用性
   - 测试工具调用

## 未来发展

### 1. Remote MCP Servers
- Cloudflare 正在开发远程 MCP 服务器支持
- 将支持基于 Web 的 MCP 客户端
- 提供更好的身份验证和授权机制

### 2. 增强集成
- 更多第三方服务集成
- 改进的工具发现机制
- 更好的错误处理和恢复

### 3. 企业功能
- 企业级安全功能
- 高级权限管理
- 审计和合规支持

## 结论

Make.com 的 MCP Server 为 AI 助手和自动化工作流程之间的集成提供了强大而简便的解决方案。通过结合 GitHub MCP Server，开发者可以创建强大的自动化系统，显著提高开发效率和项目管理能力。

关键优势包括：
- 零代码配置和部署
- 云端托管的可靠性
- 与现有工具的无缝集成
- 强大的自动化能力

随着 MCP 生态系统的持续发展，我们可以期待看到更多创新的集成方案和用例出现。

## 参考资料

1. [Model Context Protocol - Anthropic](https://modelcontextprotocol.io/)
2. [Make.com MCP Server Documentation](https://developers.make.com/mcp-server)
3. [GitHub MCP Server](https://github.com/github/github-mcp-server)
4. [Cloudflare Remote MCP Servers](https://blog.cloudflare.com/remote-model-context-protocol-servers-mcp/)
5. [Make.com MCP Server Blog Post](https://www.make.com/en/blog/model-context-protocol-mcp-server)

---

*本研究报告最后更新于 2025年6月1日*