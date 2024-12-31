---
title: 分页
---
{{< callout type="info" >}}
**协议修订**: {{< param protocolRevision >}}
{{< /callout >}}

Model Context Protocol (MCP) 支持对可能返回大量结果集的列表操作进行分页。分页允许服务器将结果分成较小的块，而不是一次性返回所有结果。

分页在通过互联网连接到外部服务时尤为重要，但对于本地集成也很有用，以避免处理大型数据集时的性能问题。

## 分页模型

MCP 中的分页使用不透明的基于游标的方法，而不是编号页。

* **游标** 是一个不透明的字符串令牌，表示结果集中的位置
* **页面大小** 由服务器确定，**可能不会** 固定

## 响应格式

分页开始时，服务器发送包含以下内容的 **响应**：
- 当前页的结果
- 如果存在更多结果，则为可选的 `nextCursor` 字段

```json
{
  "jsonrpc": "2.0",
  "id": "123",
  "result": {
    "resources": [...],
    "nextCursor": "eyJwYWdlIjogM30="
  }
}
```

## 请求格式

收到游标后，客户端可以通过包含该游标的请求 _继续_ 分页：

```json
{
  "jsonrpc": "2.0",
  "method": "resources/list",
  "params": {
    "cursor": "eyJwYWdlIjogMn0="
  }
}
```

## 分页流程

```mermaid
sequenceDiagram
    participant Client
    participant Server

    Client->>Server: 列表请求（无游标）
    loop 分页循环
      Server-->>Client: 结果页 + nextCursor
      Client->>Server: 列表请求（带游标）
    end
```

## 支持分页的操作

以下 MCP 操作支持分页：

- `resources/list` - 列出可用资源
- `resources/templates/list` - 列出资源模板
- `prompts/list` - 列出可用提示
- `tools/list` - 列出可用工具

## 实施指南

1. 服务器 **应**：
   - 提供稳定的游标
   - 优雅地处理无效游标

2. 客户端 **应**：
   - 将缺少的 `nextCursor` 视为结果的结束
   - 支持分页和非分页流程

3. 客户端 **必须** 将游标视为不透明令牌：
   - 不要对游标格式做出假设
   - 不要尝试解析或修改游标
   - 不要跨会话持久化游标

## 错误处理

无效的游标 **应** 导致代码为 -32602（无效参数）的错误。
