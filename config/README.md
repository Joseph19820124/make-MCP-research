# 配置文件说明

本文件夹包含了不同 MCP 客户端的配置模板，帮助您快速设置 Make.com 和 GitHub MCP 服务器。

## 文件说明

### `claude_desktop_config.json`
用于 Claude Desktop 的 MCP 服务器配置文件。

**使用方法:**
1. 复制此文件内容
2. 粘贴到 `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS)
3. 替换占位符：
   - `YOUR_MAKE_MCP_TOKEN_URL_HERE`: 您的 Make.com MCP Token URL
   - `YOUR_GITHUB_TOKEN_HERE`: 您的 GitHub Personal Access Token

### `vscode_mcp.json`
用于 VS Code GitHub Copilot 的 MCP 服务器配置文件。

**使用方法:**
1. 在您的项目根目录创建 `.vscode` 文件夹
2. 将此文件复制为 `.vscode/mcp.json`
3. 在 VS Code 中启用 Agent 模式时，系统会提示您输入 tokens

## 安全注意事项

⚠️ **重要提醒:**
- 绝不要将包含真实 tokens 的配置文件提交到版本控制系统
- 使用环境变量或安全的密钥管理服务存储敏感信息
- 定期轮换您的 API tokens
- 为 tokens 设置最小必要权限

## 支持的客户端

当前配置支持以下 MCP 客户端：
- Claude Desktop
- VS Code (with GitHub Copilot)
- Cursor
- Cline

## 故障排除

如果遇到配置问题，请参考主文档中的故障排除部分，或查看：
- Claude Desktop 日志: `~/Library/Logs/Claude/mcp*.log`
- VS Code 开发工具控制台输出
- Docker 容器日志: `docker logs <container-id>`