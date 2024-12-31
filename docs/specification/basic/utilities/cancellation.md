---
title: 取消
weight: 10
---

{{< callout type="info" >}}
**协议修订**: {{< param protocolRevision >}}
{{< /callout >}}

Model Context Protocol (MCP) 通过通知消息支持对正在进行的请求的可选取消。任一方都可以发送取消通知，以指示应终止先前发出的请求。

## 取消流程

当一方希望取消正在进行的请求时，它会发送一个包含以下内容的 `notifications/cancelled` 通知：

- 要取消的请求的 ID
- 可选的原因字符串，可以记录或显示

```json
{
  "jsonrpc": "2.0",
  "method": "notifications/cancelled",
  "params": {
    "requestId": "123",
    "reason": "用户请求取消"
  }
}
```

## 行为要求

1. 取消通知 **必须** 仅引用以下请求：
   - 先前在相同方向上发出的请求
   - 被认为仍在进行中的请求
2. 客户端 **不得** 取消 `initialize` 请求
3. 取消通知的接收者 **应**：
   - 停止处理被取消的请求
   - 释放相关资源
   - 不发送被取消请求的响应
4. 如果出现以下情况，接收者 **可以** 忽略取消通知：
   - 引用的请求未知
   - 处理已完成
   - 请求无法取消
5. 取消通知的发送者 **应** 忽略随后到达的请求响应

## 时间考虑

由于网络延迟，取消通知可能在请求处理完成后到达，并可能在响应已发送后到达。

双方 **必须** 优雅地处理这些竞争条件：

```mermaid
sequenceDiagram
   participant Client
   participant Server

   Client->>Server: 请求 (ID: 123)
   Note over Server: 开始处理
   Client--)Server: notifications/cancelled (ID: 123)
   alt 
      Note over Server: 处理可能在<br/>取消到达之前<br/>已完成
   else 如果未完成
      Note over Server: 停止处理
   end
```

## 实施说明

- 双方 **应** 记录取消原因以进行调试
- 应用程序 UI **应** 指示何时请求取消

## 错误处理

无效的取消通知 **应** 被忽略：

- 未知的请求 ID
- 已完成的请求
- 格式错误的通知

这保持了通知的“发送即忘”性质，同时允许在异步通信中处理竞争条件。
