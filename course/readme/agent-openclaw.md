# Agent-Openclaw

## OpenClaw 攻击面分析

{% hint style="info" %}
_\[1]YING Z, YANG X, WU S, 等. Uncovering Security Threats and Architecting Defenses in Autonomous Agents: A Case Study of OpenClaw\[A/OL]. arXiv, 2026\[2026-04-07]_
{% endhint %}

以 OpenClaw 为代表的框架赋予 AI 系统**操作系统级权限**与执行复杂工作流的自主性，这种访问权限带来了前所未有的安全挑战，传统的内容过滤防御手段已不再适用。

OpenClaw 由奥地利工程师 Peter Steinberger 开发，2025 年底以 Clawdbot 为名发布，后短暂更名为 Moltbot，2026 年 1 月定名为 OpenClaw，用户规模爆发式增长，截至 2026 年 2 月底 GitHub 星标超 20 万。行业分析师称其为 “个人 AI 助手的未来”，但福布斯、CNET 等安全专家同时提醒，该框架仍处于早期阶段，**缺乏企业级合规防护**，安全能力不成熟的机构部署风险极高。

OpenClaw（俗称 “龙虾”）是一款流行的开源、自托管自主 AI 虚拟助手，openclaw的特点为持久、面向操作的数字代理，可在本地操作系统与外部云服务间执行复杂多步工作流。如图所示，OpenClaw 采用**解耦、本地优先架构**，通过多个互联子系统将 LLM 认知能力与深度系统级执行能力打通

<figure><img src="../../.gitbook/assets/image (111).png" alt=""><figcaption></figcaption></figure>

OpenClaw 的优势源于**解耦、本地优先架构**，将 LLM 认知能力与系统级执行深度结合，核心子系统包括：

* **网关（控制面）**：本地优先 WebSocket 网络，作为集中控制平面，管理用户会话、在线状态、定时任务、Webhook，并将入站事件路由到隔离的智能体工作区。
* **Pi 智能体运行时**：核心推理引擎，采用 RPC 模式，支持工具流与块流；本地存储配置数据与交互历史以维护持久记忆，使智能体跨会话保持长期上下文。
* **全渠道集成**：作为统一收发箱，无缝集成 20 余种通信渠道，包括企业平台（Slack、Teams、飞书、Mattermost）与个人通讯工具（WhatsApp、Telegram、Signal、Discord、iMessage）。
* **跨平台节点**：在 macOS、iOS、Android 部署原生节点，赋予智能体访问设备专属硬件与操作系统 API 的能力，包括相机控制、屏幕录制、定位服务、直接执行系统命令（如 macOS 的 system.run、system.notify）。
* **高级工具与自动化**：自带由 Chrome 开发工具协议（CDP）控制的专属浏览器实例，支持自主网页浏览交互；提供 “实时画布” 用于可视化工作区操作，以及健壮的第三方插件集成技能平台。

OpenClaw 的革命性生产力特性，同时造就了**前所未有的攻击面**。其设计使其连接真实、面向公网的消息界面，虽配备配对码等基础安全策略防止未授权访问，但智能体仍持续暴露在互联网不可信输入中。此外，Openclaw可自主调用浏览器、执行 Shell 命令、读写本地文件，意味着成功的提示注入或工具劫持攻击不仅会产生错误信息，还会导致**任意代码执行、内网横向移动、严重数据泄露**。

