---
title: 进度
weight: 30
---
{{< callout type="info" >}}
**协议修订**: {{< param protocolRevision >}}
{{< /callout >}}

Model Context Protocol (MCP) 通过通知消息支持对长时间运行操作的可选进度跟踪。任一方都可以发送进度通知，以提供有关操作状态的更新。

## 进度流程

当一方希望 _接收_ 请求的进度更新时，它会在请求元数据中包含一个 `progressToken`。

* 进度令牌 **必须** 是字符串或整数值
* 进度令牌可以由发送者通过任何方式选择，但 **必须** 在所有活动请求中唯一。

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "some_method",
  "params": {
    "_meta": {
      "progressToken": "abc123"
    }
  }
}
```

接收者 **可以** 然后发送包含以下内容的进度通知：

- 原始进度令牌
- 到目前为止的当前进度值
- 可选的“总计”值

```json
{
  "jsonrpc": "2.0",
  "method": "notifications/progress",
  "params": {
    "progressToken": "abc123",
    "progress": 50,
    "total": 100
  }
}
```

* 即使总数未知，`progress` 值 **必须** 随每个通知增加。
* `progress` 和 `total` 值 **可以** 是浮点数。

## 行为要求

1. 进度通知 **必须** 仅引用以下令牌：
   - 在活动请求中提供的令牌
   - 与正在进行的操作相关联的令牌

2. 进度请求的接收者 **可以**：
   - 选择不发送任何进度通知
   - 以其认为合适的频率发送通知
   - 如果未知，则省略总值

```mermaid
sequenceDiagram
    participant Sender
    participant Receiver

    Note over Sender,Receiver: 带有进度令牌的请求
    Sender->>Receiver: 带有 progressToken 的方法请求

    Note over Sender,Receiver: 进度更新
    loop 进度更新
        Receiver-->>Sender: 进度通知 (0.2/1.0)
        Receiver-->>Sender: 进度通知 (0.6/1.0)
        Receiver-->>Sender: 进度通知 (1.0/1.0)
    end

    Note over Sender,Receiver: 操作完成
    Receiver->>Sender: 方法响应
```

## 实施说明

- 发送者和接收者 **应** 跟踪活动进度令牌
- 双方 **应** 实施速率限制以防止泛滥
- 进度通知 **必须** 在完成后停止
