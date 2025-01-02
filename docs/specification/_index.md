---
title: 规范
cascade:
  type: docs
breadcrumbs: false
weight: 10
---

{{< callout type="info" >}}
**协议修订**: {{< param protocolRevision >}}
{{< /callout >}}

[Model Context Protocol](https://model-context-protocol.mintlify.app) (MCP) 是一个开放协议，能够在 LLM 应用程序和外部数据源及工具之间实现无缝集成。无论您是在构建 AI 驱动的 IDE、增强聊天界面，还是创建自定义 AI 工作流，MCP 都提供了一种标准化的方式来连接 LLM 与所需的上下文。

本规范定义了权威的协议要求，基于 [schema.ts](https://github.com/modelcontextprotocol/specification/blob/main/schema/schema.ts) 中的 TypeScript 模式。

有关实施指南和示例，请访问 [model-context-protocol.mintlify.app](https://model-context-protocol.mintlify.app)。

本文件中的关键字 "MUST"、"MUST NOT"、"REQUIRED"、"SHALL"、"SHALL NOT"、"SHOULD"、"SHOULD NOT"、"RECOMMENDED"、"NOT RECOMMENDED"、"MAY" 和 "OPTIONAL" 应按照 [BCP 14](https://datatracker.ietf.org/doc/html/bcp14) [[RFC2119](https://datatracker.ietf.org/doc/html/rfc2119)] [[RFC8174](https://datatracker.ietf.org/doc/html/rfc8174)] 中的描述进行解释，仅当它们以全大写形式出现时，如此处所示。

## 概述

MCP 提供了一种标准化的方式，使应用程序能够：

- 与语言模型共享上下文信息
- 向 AI 系统公开工具和功能
- 构建可组合的集成和工作流

该协议使用 [JSON-RPC](https://www.jsonrpc.org/) 2.0 消息在以下各方之间建立通信：

- **Hosts**: 发起连接的 LLM 应用程序
- **Clients**: 主机应用程序中的连接器
- **Servers**: 提供上下文和功能的服务

MCP 从 [Language Server Protocol](https://microsoft.github.io/language-server-protocol/) 中汲取了一些灵感，该协议标准化了如何在整个开发工具生态系统中添加编程语言支持。类似地，MCP 标准化了如何将额外的上下文和工具集成到 AI 应用程序的生态系统中。

## 关键细节

### 基础协议
- [JSON-RPC](https://www.jsonrpc.org/) 消息格式
- 有状态连接
- 服务器和客户端功能协商

### 功能

服务器向客户端提供以下任何功能：

- **资源**: 上下文和数据，供用户或 AI 模型使用
- **提示**: 为用户提供的模板消息和工作流
- **工具**: 供 AI 模型执行的功能

客户端可以向服务器提供以下功能：

- **采样**: 服务器发起的代理行为和递归 LLM 交互

### 其他实用工具

- 配置
- 进度跟踪
- 取消
- 错误报告
- 日志记录

## 安全和信任与安全

Model Context Protocol 通过任意数据访问和代码执行路径提供强大的功能。随着这种能力的增强，所有实施者必须仔细解决重要的安全和信任问题。

### 关键原则

1. **用户同意和控制**
   - 用户必须明确同意并理解所有数据访问和操作
   - 用户必须保留对共享数据和采取行动的控制权
   - 实施者应提供清晰的用户界面，用于审查和授权活动

2. **数据隐私**
   - 主机必须在向服务器公开用户数据之前获得用户的明确同意
   - 主机不得在未经用户同意的情况下传输资源数据
   - 用户数据应受到适当的访问控制保护

3. **工具安全**
   - 工具代表任意代码执行，必须谨慎对待
   - 主机必须在调用任何工具之前获得用户的明确同意
   - 用户应在授权使用工具之前了解每个工具的功能

4. **LLM 采样控制**
   - 用户必须明确批准任何 LLM 采样请求
   - 用户应控制：
     - 是否进行采样
     - 将发送的实际提示
     - 服务器可以看到的结果
   - 协议有意限制了服务器对提示的可见性

### 实施指南

虽然 MCP 本身无法在协议级别强制执行这些安全原则，但实施者 **应**：

1. 在其应用程序中构建健全的同意和授权流程
2. 提供清晰的安全影响文档
3. 实施适当的访问控制和数据保护
4. 在其集成中遵循安全最佳实践
5. 在其功能设计中考虑隐私影响

## 了解更多

探索每个协议组件的详细规范：

{{< cards >}}
  {{< card link="architecture" title="架构" icon="template" >}}
  {{< card link="basic" title="基础协议" icon="code" >}}
  {{< card link="server" title="服务器功能" icon="server" >}}
  {{< card link="client" title="客户端功能" icon="user" >}}
  {{< card link="contributing" title="贡献" icon="pencil" >}}
{{< /cards >}}
