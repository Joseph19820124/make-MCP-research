# 快速开始指南

## 1. 设置 Make.com MCP Server

### 步骤 1: 准备 Make.com 场景
1. 在 Make.com 中创建新场景
2. 将场景设置为"按需"运行
3. 确保场景是"激活"状态
4. 配置适当的输入和输出参数

### 步骤 2: 获取 MCP Token
1. 登录 Make.com
2. 进入用户配置文件 → API/MCP 访问
3. 点击"添加 token" → 选择"MCP Token"
4. 复制生成的 URL

### 步骤 3: 配置 Claude Desktop
编辑 `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "make": {
      "command": "npx",
      "args": ["-y", "@makehq/mcp-server"],
      "env": {
        "MAKE_MCP_TOKEN_URL": "你复制的MCP_URL"
      }
    }
  }
}
```

## 2. 设置 GitHub MCP Server

### 步骤 1: 创建 GitHub Personal Access Token
1. 访问 GitHub Settings → Developer settings → Personal access tokens
2. 生成新 token，设置必要权限：
   - `repo` (仓库访问)
   - `issues` (问题管理)
   - `pull_requests` (PR管理)

### 步骤 2: 配置 Claude Desktop
将以下内容添加到配置文件：

```json
{
  "mcpServers": {
    "make": {
      "command": "npx",
      "args": ["-y", "@makehq/mcp-server"],
      "env": {
        "MAKE_MCP_TOKEN_URL": "你的MCP_URL"
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
        "GITHUB_PERSONAL_ACCESS_TOKEN": "你的GitHub_TOKEN"
      }
    }
  }
}
```

### 步骤 3: 重启 Claude Desktop
重启应用以加载新配置。

## 3. 验证设置

在 Claude Desktop 中测试：

```
请帮我创建一个 GitHub issue 来跟踪这个 bug
```

或

```
请运行我的 Make 场景来处理用户反馈
```

## 故障排除

### 常见问题
1. **MCP 服务器未显示**: 检查 JSON 配置格式
2. **连接失败**: 验证 token 有效性
3. **权限错误**: 检查 GitHub PAT 权限设置

### 调试命令
```bash
# 查看 Claude 日志
tail -f ~/Library/Logs/Claude/mcp*.log

# 测试 Docker 镜像
docker run -i --rm -e GITHUB_PERSONAL_ACCESS_TOKEN=your_token ghcr.io/github/github-mcp-server
```