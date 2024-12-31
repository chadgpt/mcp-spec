---
title: 基础协议
cascade:
  type: docs
weight: 2
---

{{< callout type="info" >}}
**协议修订**: {{< param protocolRevision >}}
{{< /callout >}}

所有 MCP 客户端和服务器之间的消息 **必须** 遵循 [JSON-RPC 2.0](https://www.jsonrpc.org/specification) 规范。该协议定义了三种基本类型的消息：

| 类型           | 描述                                    | 要求                                      |
|----------------|----------------------------------------|-------------------------------------------|
| `Requests`     | 用于启动操作的消息                      | 必须包含唯一 ID 和方法名称                |
| `Responses`    | 回复请求的消息                          | 必须包含与请求相同的 ID                   |
| `Notifications`| 无需回复的单向消息                      | 不得包含 ID                               |

**Responses** 进一步分为 **成功结果** 或 **错误**。结果可以遵循任何 JSON 对象结构，而错误必须至少包含错误代码和消息。

## 协议层

Model Context Protocol 由多个关键组件组成，这些组件协同工作：

- **基础协议**: 核心 JSON-RPC 消息类型
- **生命周期管理**: 连接初始化、功能协商和会话控制
- **服务器功能**: 服务器公开的资源、提示和工具
- **客户端功能**: 客户端提供的采样和根目录列表
- **实用工具**: 跨领域关注点，如日志记录和参数完成

所有实现 **必须** 支持基础协议和生命周期管理组件。其他组件 **可以** 根据应用程序的具体需求实现。

这些协议层建立了明确的关注点分离，同时使客户端和服务器之间的丰富交互成为可能。模块化设计允许实现仅支持所需的功能。

有关不同组件的更多详细信息，请参见以下页面：

{{< cards >}}
  {{< card link="/specification/basic/lifecycle" title="生命周期" icon="refresh" >}}
  {{< card link="/specification/server/resources" title="资源" icon="document" >}}
  {{< card link="/specification/server/prompts" title="提示" icon="chat-alt-2" >}}
  {{< card link="/specification/server/tools" title="工具" icon="adjustments" >}}
  {{< card link="/specification/server/utilities/logging" title="日志记录" icon="annotation" >}}
  {{< card link="/specification/client/sampling" title="采样" icon="code" >}}
{{< /cards >}}

## 认证

认证和授权目前不是 MCP 核心规范的一部分，但我们正在考虑在未来引入它们。加入我们的 [GitHub 讨论](https://github.com/modelcontextprotocol/specification/discussions)，帮助塑造协议的未来！

客户端和服务器 **可以** 协商自己的自定义认证和授权策略。

## 模式

协议的完整规范定义为 [TypeScript 模式](http://github.com/modelcontextprotocol/specification/tree/main/schema/schema.ts)。这是所有协议消息和结构的真实来源。

还有一个 [JSON 模式](http://github.com/modelcontextprotocol/specification/tree/main/schema/schema.json)，它是从 TypeScript 真实来源自动生成的，用于各种自动化工具。
