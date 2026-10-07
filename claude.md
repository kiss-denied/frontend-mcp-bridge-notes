# Claude 接入说明

记录日期：2026-10-08。以下是依据官方文档整理的接入方向，本项目还没有完成 Claude 实机验收。

## 路径一：Claude Desktop 的本地 MCP

如果希望先在同一台电脑测试，可使用 Claude Desktop 支持的本地 MCP 配置或扩展。已有 stdio 适配器可以作为连接基础，但仍需确认客户端启动方式、依赖和权限。

一个适合本项目的适配器设计是：通过标准输入输出接收 MCP 消息，在本机向 Bridge 转发；所需认证信息由本机安全存储读取。标准输出只发送协议消息，诊断输出放到标准错误，避免破坏 MCP 通信。

先读取一小段允许访问的状态，再测试一个明确授权的写操作，最后检查前端是否显示变化。不要用批量日记导出来测试连通性。

官方参考：[Claude Desktop 本地 MCP 入门](https://support.claude.com/en/articles/10949351-getting-started-with-local-mcp-servers-on-claude-desktop)。

## 路径二：Claude 远程自定义连接

Claude 的远程连接由 Anthropic 云端访问 MCP 服务。即使使用 Desktop 添加远程连接，也需要云端可达的服务地址；用户电脑的 localhost 不能直接提供这个入口。

因此需要另外准备适合 Claude 的远程访问方式，例如带认证的 HTTPS MCP 服务，并检查：

1. 服务传输方式符合当前客户端要求。
2. 域名、证书与网络可达性正常。
3. 客户端支持服务采用的认证流程。
4. 后端将连接绑定到预期身份。
5. 真实工具调用与前端变化都通过验收。

不要为了连通性移除现有认证。公开仓库的访问地址也不是 MCP 服务地址。

官方参考：[远程 MCP 自定义连接入门](https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp)。

## OpenAI 隧道能直接给 Claude 用吗？

本项目没有验证这种用法。[OpenAI Secure MCP Tunnel 官方文档](https://developers.openai.com/api/docs/guides/secure-mcp-tunnels)说明的是 OpenAI 产品访问私有 MCP 服务的方式。不能据此推断 Claude 能消费同一个隧道标识，也不能把该标识当作通用 HTTPS endpoint。

可以复用的是工具定义、后端业务逻辑、数据与授权模型。连接入口需按各客户端实际支持的机制适配。

## 如果也使用 Claude Code

Claude Code 支持 MCP 配置，包括本地 stdio 和远程 HTTP 等方式；它与 Claude 网页端的远程连接设置不是同一个入口。具体语法以 [Claude Code 官方 MCP 文档](https://code.claude.com/docs/en/mcp)为准。

## 一定先决定身份

如果 ChatGPT 与 Claude 对应同一个业务身份，就明确复用该身份允许的权限。如果希望两个模型是不同角色，就在后端建立独立身份绑定与授权。

模型在对话里声称自己是谁，不构成访问权限。界面和 MCP 工具都应由同一个后端检查身份。

连接成功后，依次验证工具发现、授权读取、授权写入、前端同步、撤销授权后的拒绝行为。只有这些通过，才能说这条 Claude 路径在自己的项目里成功。

