---
title: 版本控制
type: docs
weight: 80
---

Model Context Protocol 使用基于字符串的版本标识符，格式为 `YYYY-MM-DD`，以指示最后一次进行向后不兼容更改的日期。

当前协议版本为 **{{< param protocolRevision >}}**。 [查看所有修订]({{< ref "/specification/revisions" >}})。

{{< callout type="info" >}}
  只要协议更新保持向后兼容，协议版本将 _不会_ 增加。这允许在保持互操作性的同时进行增量改进。
{{< /callout >}}

版本协商发生在 [初始化]({{< ref "/specification/basic/lifecycle#initialization" >}}) 期间。客户端和服务器 **可以** 同时支持多个协议版本，但它们 **必须** 就会话中使用的单一版本达成一致。

如果版本协商失败，协议提供适当的错误处理，允许客户端在无法找到与服务器兼容的版本时优雅地终止连接。
