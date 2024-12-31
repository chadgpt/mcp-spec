---
title: 采样
type: docs
weight: 40
---

{{< callout type="info" >}}
**协议修订**: {{< param protocolRevision >}}
{{< /callout >}}

Model Context Protocol (MCP) 提供了一种标准化方式，使服务器能够通过客户端请求 LLM 采样（“完成”或“生成”）。此流程允许客户端保持对模型访问、选择和权限的控制，同时使服务器能够利用 AI 功能&mdash;无需服务器 API 密钥。服务器可以请求基于文本或图像的交互，并可选择在其提示中包含来自 MCP 服务器的上下文。

## 用户交互模型

MCP 中的采样允许服务器实现代理行为，通过使 LLM 调用 _嵌套_ 在其他 MCP 服务器功能中来实现。

实现可以通过任何适合其需求的界面模式公开采样&mdash;协议本身不强制规定任何特定的用户交互模型。

{{< callout type="warning" >}}
  出于信任与安全和安全考虑，**应** 始终有一个人在循环中，能够拒绝采样请求。
  
  应用程序 **应**：
  * 提供易于直观地审查采样请求的 UI
  * 允许用户在发送前查看和编辑提示
  * 在交付前呈现生成的响应以供审查
{{< /callout >}}

## 功能

支持采样的客户端 **必须** 在 [初始化]({{< ref "/specification/basic/lifecycle#initialization" >}}) 期间声明 `sampling` 功能：

```json
{
  "capabilities": {
    "sampling": {}
  }
}
```

## 协议消息

### 创建消息

要请求语言模型生成，服务器发送 `sampling/createMessage` 请求：

**请求:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "sampling/createMessage",
  "params": {
    "messages": [
      {
        "role": "user",
        "content": {
          "type": "text",
          "text": "法国的首都是哪里？"
        }
      }
    ],
    "modelPreferences": {
      "hints": [
        {
          "name": "claude-3-sonnet"
        }
      ],
      "intelligencePriority": 0.8,
      "speedPriority": 0.5
    },
    "systemPrompt": "你是一个乐于助人的助手。",
    "maxTokens": 100
  }
}
```

**响应:**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "role": "assistant",
    "content": {
          "type": "text",
          "text": "法国的首都是巴黎。"
    },
    "model": "claude-3-sonnet-20240307",
    "stopReason": "endTurn"
  }
}
```

## 消息流程

```mermaid
sequenceDiagram
    participant Server
    participant Client
    participant User
    participant LLM

    Note over Server,Client: 服务器启动采样
    Server->>Client: sampling/createMessage

    Note over Client,User: 人在循环中审查
    Client->>User: 提出请求以供批准
    User-->>Client: 审查并批准/修改

    Note over Client,LLM: 模型交互
    Client->>LLM: 转发批准的请求
    LLM-->>Client: 返回生成

    Note over Client,User: 响应审查
    Client->>User: 提出响应以供批准
    User-->>Client: 审查并批准/修改

    Note over Server,Client: 完成请求
    Client-->>Server: 返回批准的响应
```

## 数据类型

### 消息

采样消息可以包含：

#### 文本内容
```json
{
  "type": "text",
  "text": "消息内容"
}
```

#### 图像内容
```json
{
  "type": "image",
  "data": "base64 编码的图像数据",
  "mimeType": "image/jpeg"
}
```

### 模型偏好

MCP 中的模型选择需要仔细抽象，因为服务器和客户端可能使用不同的 AI 提供商，提供不同的模型。服务器不能简单地按名称请求特定模型，因为客户端可能无法访问该模型，或者可能更喜欢使用不同提供商的等效模型。

为了解决这个问题，MCP 实现了一个偏好系统，将抽象能力优先级与可选的模型提示相结合：

#### 能力优先级

服务器通过三个归一化的优先级值（0-1）表达其需求：

- `costPriority`: 最小化成本的重要性？较高的值偏向于更便宜的模型。
- `speedPriority`: 低延迟的重要性？较高的值偏向于更快的模型。
- `intelligencePriority`: 高级功能的重要性？较高的值偏向于更强大的模型。

#### 模型提示

虽然优先级有助于根据特性选择模型，但 `hints` 允许服务器建议特定模型或模型系列：

- 提示被视为子字符串，可以灵活地匹配模型名称
- 多个提示按优先顺序评估
- 客户端 **可以** 将提示映射到不同提供商的等效模型
- 提示是建议性的&mdash;客户端做出最终的模型选择

例如:
```json
{
  "hints": [
    {"name": "claude-3-sonnet"},  // 更喜欢 Sonnet 类模型
    {"name": "claude"}            // 回退到任何 Claude 模型
  ],
  "costPriority": 0.3,            // 成本不太重要
  "speedPriority": 0.8,           // 速度非常重要
  "intelligencePriority": 0.5     // 中等能力需求
}
```

客户端处理这些偏好，从其可用选项中选择适当的模型。例如，如果客户端无法访问 Claude 模型，但有 Gemini，它可能会根据类似的能力将 sonnet 提示映射到 `gemini-1.5-pro`。

## 错误处理

客户端 **应** 返回常见故障情况的错误：

错误示例:
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "error": {
    "code": -1,
    "message": "用户拒绝了采样请求"
  }
}
```

## 安全考虑

1. 客户端 **应** 实施用户批准控制
2. 双方 **应** 验证消息内容
3. 客户端 **应** 尊重模型偏好提示
4. 客户端 **应** 实施速率限制
5. 双方 **必须** 适当处理敏感数据
