---
description: 智能体协议
cover: ../../.gitbook/assets/微信图片_20250408134702.jpg
coverY: 108
---

# MCP & A2A

## MCP,**Model Context Protocol**

MCP（Model Context Protocol，模型上下文协议）由Anthropic于2024年11月推出，是一套开放协议标准，旨在规范AI模型与外部数据源、工具之间的交互方式

{% embed url="https://blog.csdn.net/fufan_LLM/article/details/146377471" %}

核心目标是解决以下问题：

* **数据孤岛：**&#x6A21;型无法直接访问实时数据或本地资源；
* **集成复杂性：**&#x4E3A;每个工具编写独立接口导致开发成本高；
* **生态碎片化：**&#x4E0D;同平台的工具调用机制缺乏统一标准；
* **安全隐患：**&#x7F3A;乏标准化的访问控制机制

核心架构：MCP采用客户端-服务器架构，包含三个组件

* 主机（Host）：运行LLM的应用程序（如Claude Desktop、IDE插件）；
* 客户端（Client）：与服务器建立连接，管理会话和路由消息；
* 服务器（Server）：提供资源访问、工具调用等能力的独立服务。

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

根据 MCP 的规范，当前支持两种传输方式：标准输入输出（stdio）和基于 HTTP 的服务器推送事件（SSE）。而近期，开发者在 MCP 的 GitHub 仓库中提交了一项提案，建议采用“可流式传输的 HTTP”来替代现有的 HTTP+SSE 方案。此举旨在解决当前远程 MCP 传输方式的关键限制，同时保留其优势。 HTTP 和 SSE（服务器推送事件）在数据传输方式上存在明显区别：

* **通信方式**
  * HTTP：采用请求-响应模式，客户端发送请求，服务器返回响应，每次请求都是独立的。
  * SSE：允许服务器通过单个持久的 HTTP 连接，持续向客户端推送数据，实现实时更新。
* 连接特性：
  * HTTP：每次请求通常建立新的连接，虽然在 HTTP/1.1 中引入了持久连接，但默认情况下仍是短连接。
  * SSE：基于长连接，客户端与服务器之间保持持续的连接，服务器可以在任意时间推送数据。
* 适用场景：
  * HTTP：适用于传统的请求-响应场景，如网页加载、表单提交等。
  * SSE：适用于需要服务器主动向客户端推送数据的场景，如实时通知、股票行情更新等。

需要注意的是，SSE 仅支持服务器向客户端的单向通信，而 WebSocket 则支持双向通信。

本地通讯：使用了stdio传输数据，具体流程Client启动Server程序作为子进程，其消息通讯是通过stdin/stdout进行的，消息格式为JSON-RPC 2.0。

远程通讯：Client与Server可以部署在任何地方，Client使用SSE与Server进行通讯，消息的格式为JSON-RPC 2.0，Server定义了/see与/messages接口用于推送与接收数据。\
\


