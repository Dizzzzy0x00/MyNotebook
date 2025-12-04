---
layout:
  width: wide
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
---

# iotFuzz-3

## IP摄像头

### Shodan

{% embed url="https://www.shodan.io/" %}

识别互联网网络上暴露的摄像头

{% embed url="https://bugprove.com/knowledge-hub/cve-2023-3959-cve-2023-4249-multiple-critical-vulnerabilities-in-zavio-ip-cameras/" %}

### Ientifying Challenges with Fuzzing the RLC-510WA IP Camera using AFL++

{% embed url="https://ntnuopen.ntnu.no/ntnu-xmlui/handle/11250/3204628" %}

总体遵循 OWASP 的 Firmware Security Testing Methodology（FSTM），使用AFL++的QEMU模式对 RLC-510WA的智能安全IP摄像头进行模糊测试，**并未发现任何crash或者安全漏洞**

* **固件获取**：使用 UART（Bus Pirate）连接并通过已有研究人员提供的 root credentials 获取 shell，从而提取固件与文件系统
* **文件系统提取 & 目标识别**：采用 binwalk/UBI 工具等提取文件系统，并定位可能作为 fuzz 目标的服务二进制（ONVIF、RTSP、NetServer、CGI 等）。
* **模拟（Emulation）**：使用 QEMU（vexpress 等平台）进行系统级模拟。因为没有 100% 相同的开发板，他们使用“相似板”并手工构建内核与 DTB，复制文件系统到可挂载镜像并多次调试启动脚本（init-wrapper 等）以让服务在 QEMU 中运行。
* **动态分析 / AFL++ 配置**：
  * 在宿主上安装 AFL++，尝试两种模式：**非 instrumented（无覆盖引导）** 的网络 fuzz（send-to-onvif.sh）与 **instrumented**（QEMU 模式）运行（论文展示了构建 AFL++ QEMU 模式、缺失包、修复 symlink 的操作）。他们发现非 instrumented 模式可行但效果差，instrumented 模式依赖文件系统布局、库符号等，容易出现兼容问题。
  * 进行了网络层黑盒 fuzz（使用 XML 字典与 send 脚本）并尝试通过 AFL++（非 instrumented）跑 PoC。也尝试了用编译好的 C-wrapper 替代 Bash wrapper，这样可显著提高执行速度。
  * 没有发现可复现的 crash/漏洞（因此也没进入 runtime 分析 / 利用阶段）。论文说明未发现挂起/崩溃，无法做进一步利用

### Camveil: Unveiling Security Camera Vulnerabilities through Multi-Protocol&#xD; Coordinated Fuzzing

{% embed url="https://www.computer.org/csdl/proceedings-article/sp/2026/606500a021/2bojv6StLlS" %}

在实际部署中，摄像机协议通常通过访问和修改共享的系统状态进行交互。当多个客户端同时通过不同的协议发出操作公共内部资源的命令时，就会出现这种交互。例如，如图 1 所示，HTTP 客户端可以发送 `continuous_move` 请求，将摄像机的运动状态从空闲更改为运动状态，并更新其位置值。与此同时，ONVIF 客户端可以发出 `move_up` 命令，同样修改摄像机的运动和位置状态。类似地，RTSP 客户端可以通过 `PLAY` 请求启动视频流，而 ONVIF 客户端同时重置编码方式，这两个操作都会影响摄像机的编码状态

<figure><img src="../.gitbook/assets/80f3bbe90ac6e7b8535435507ff67534.png" alt=""><figcaption></figcaption></figure>

* **ONVIF**：跨厂商安全设备标准协议。
* **RTSP**：实时流媒体控制（PLAY、PAUSE、SETUP）。
* **HTTP**：用于 PTZ 控制和各种配置项（如速度、位置）。



一个漏洞示例：[CVE2023-3959](https://nvd.nist.gov/vuln/detail/CVE-2023-3959)，并发的ONVIF和RTSP请求访问共享资源（例如编码状态）时，该漏洞会被触发，导致XML解析错误和实时视频流中断。

{% embed url="https://www.iotsec-zone.com/article/423" %}

<figure><img src="../.gitbook/assets/3cb927f161f32fb11226672002485717.png" alt=""><figcaption></figcaption></figure>

第一步：攻击者发送恶意 ONVIF 配置请求数据包包含如`SetVideoEncoderConfiguration` / `SetSystemDateAndTime` 或类似的 ONVIF 配置操作的字段

* 这些字段允许字符串内容（如 Encoder name、Profile name）。
* 字段内容未严格验证。
* 攻击者塞入恶意 payload，例如：`$(sleep 9999)` 或者`; reboot;`

ONVIF 守护进程把这些字段写入一个 **全局结构体 global\_config\_struct** 里。**这个结构体稍后会被其他模块读取。**

第二步：攻击者发送另一个 HTTP 请求

* 访问 Web 管理界面 CGI，如\
  `/param.cgi?action=update&...`
* 此操作会调用 firmware 中的函数（论文说是 `check_para()`）。
  * 它会从之前的**同一个全局结构体**中读取 encoder 配置字段。
  * 然后执行类似：`popen(global_config_struct->encoder_name, "r");`
* 如果这个字段是：`$(sleep 9999)` 或`; busybox reboot ;`，就会**直接通过 popen 执行命令**。

<table><thead><tr><th width="124.33331298828125">动作</th><th>协议</th><th>行为</th></tr></thead><tbody><tr><td>1</td><td><strong>ONVIF</strong></td><td>写入恶意数据到全局结构</td></tr><tr><td>2</td><td><strong>HTTP</strong></td><td>触发另一个进程逻辑读取这份数据</td></tr><tr><td>3</td><td><strong>HTTP</strong></td><td>popen 执行数据 → RCE</td></tr></tbody></table>

设计目标：一个实用的安全摄像头漏洞检测框架应具备以下特性：

* 跨协议交互覆盖。该框架应能有效探索跨多个协议的交互。许多实际应用中的IP摄像头漏洞都源于不同协议之间的相互作用。系统必须能够生成语义清晰且状态感知的消息序列，以反映真实的多协议工作流程并暴露交互引起的漏洞。
* 检测非崩溃逻辑漏洞。该框架应能够识别不会导致系统崩溃的功能性漏洞。这些漏洞包括命令注入、配置不一致和视频流冻结。为了检测此类细微问题，系统必须集成语义监视器（VLC播放器、PTZ状态检索器和系统状态检索器生成标准请求来查询摄像机的运行时状态），以分析运行时行为并检测除简单连接丢失或异常之外的异常情况。
* 黑盒兼容性。由于许多商用IP摄像头提供有限的访问权限和闭源固件，该框架必须在黑盒环境下工作，在这种情况下收集代码覆盖率是不可行的。它应仅依赖于外部可观察的行为和公开的协议接口，而无需插桩、调试符号或root权限。

<figure><img src="../.gitbook/assets/6f37a5afc2c6dbefc49a918cb8f8b59c.png" alt=""><figcaption></figcaption></figure>

## NVR设备

{% embed url="https://www.cve.org/CVERecord/SearchResults?query=NVR" %}

<table><thead><tr><th width="163">类别</th><th>监控内容</th></tr></thead><tbody><tr><td>system</td><td>进程、CPU、内存、重启</td></tr><tr><td>user</td><td>用户管理文件、账号更改</td></tr><tr><td>storage</td><td>HDD 状态、录像文件</td></tr><tr><td>network</td><td>端口、cloud 连接</td></tr><tr><td>video</td><td>RTSP 通道（多个通道）</td></tr></tbody></table>

#### **无 Crash 的逻辑漏洞**&#x20;

这种漏洞不会导致：

* 崩溃
* 异常退出
* 内存越界

但会导致

权限绕过、弱权限检查、文件未授权读取、配置被默默修改、命令被执行但返回数据正常 如200 OK、越权 API 调用

#### **Side-Effect（系统副作用）触发的漏洞 —— 系统产生异常行为**

很常见于 NVR / DVR / 路由器：

* 发一个请求 → 返回 200
* 但后台执行了危险操作

比如：

* 创建新用户
* 修改某配置
* 触发守护进程异常
* 导致录像停止
* 删除录像文件
* 写入某路径
* 导致系统重启或卡死（但不 crash）
* 接口执行耗时命令（CPU 满负载）

#### **配置变更引发的漏洞（Configuration-driven Bugs）**

很多 IoT / NVR 的漏洞不仅与输入有关，还与设备配置状态有关：

举例：

* 勾选某项 → 某个 CGI 自动被激活
* 开启“云同步” → 内部守护进程启动并开放未认证接口
* 修改语言 → CGI 使用不同编码 → 引发解析错误
* 切换视频通道 → 触发某个旧版本函数

很多漏洞必须在**特定设置下**才能复现。普通 fuzz 不会改变设备配置，因此无法测试。

### IotHunter:面向物联网固件有状态协议的模糊测试工具,CCF-A

{% embed url="https://dl.acm.org/doi/pdf/10.1145/3319535.3363247" %}

在物联网固件的运行监控过程中， IoTHunter 根据给定的状态序列顺序切换到协议状态，进行基于反馈的状态探索，通过多级消息生成实现覆盖引导的灰盒模糊。IoTHunter 在功能覆盖、块覆盖和边缘覆盖方面分别比黑盒模糊优 2.2 倍、2.0 倍 和 2.5 倍。并且它支持物联网固件中的多个关键协议(如 snmp、 ftp、 ssl、 bgp、 smb)，IoTHunter 还在家庭路由器 Mikrotik 的固件中发现了 五个新的漏洞。

IOTHunter设计的模型 $$G=（S，M）$$（如下图所示）， $$S =< s_0, s_1, s_2, …, s_n >$$指不同的状态， $$M = \{M_{i, j}, 0\le i, j \le n \}$$是消息集合。每个消息类型 $$M_{i, j}$$表示可以从状态 si 迁移到状态 sj 的消息。特别是，如果 j = 0，我们称$$M_{i, 0}$$ 为无效消息，这将导致系统丢失当前状态 si 并迁移到初始状态 s0。实心箭头和虚线箭头分别表示有效和无效的消息类型

在有状态模糊过程中，测试用例是模糊器生成的新消息。**消息是否有效取决于物联网固件的运行状态监控。**&#x56E0;此，在本质上，有状态模糊过程是找到输入空间的有效和无效区域之间的边界。

<figure><img src="../.gitbook/assets/ae3e5929a012044621f298787aa1ac5a.png" alt=""><figcaption></figcaption></figure>

* 输入:输入由两部分组成：1、状态序列S和能够遍历状态的信息序列M；2、报文规范
* 工作流:
  * 首先使用特定于协议的解析器根据格式要求检查每个消息来提取元数据。然后，根据状态序列S调度协议状态，调用消息突变调度模块生成新的基于消息种子的测试用例。在那之后，我们根据消息格式需求检查测试用例，如果发生任何违规，则更新相应的字段。最后，将消息发送到物联网固件的运行目标，以收集执行跟踪和消息响应。
* 协议状态调度过程:通过考虑消息序列中的已知状态和未知状态，协议状态调度过程由两部分组成：
  * 已知状态模糊: 从初始状态 s0 开始，基于消息种子直接生成新的消息。然后，当任何违反格式要求的情况发生时更新消息格式。在将新消息发送到目标物联网固件之后，如果新消息有效，那么将实现下一个状态 s1，并且我们收集有效消息 $$M_{0, 1}$$。否则，当前状态将丢失并重置为 s0。经过一段时间的模糊处理后，如果没有找到新的路径，将继续进入下一个状态。对于每个步骤中的状态 si (i > 0) 的模糊化，首先需要检查当前状态，如果不是 si-1，则从有效消息集 中选择附加消息 $$Mi = m_{0,1} + M_{1,2} + ... + M_{i-2}$$，i-1 用于将当前状态调度到 si-1
  * 未知状态模糊：在变异过程中，如果能发现状态序列中没有的状态，就把当前消息序列以及状态序列加入输入库之中。
* 格式检测:在fuzzing过程中，每个消息在发送之前根据协议格式要求进行检查。基于特定于协议的解析器，提取并存储每条消息的元数据。然后，检查元数据是否保留消息验证。例如，新消息的长度被重新计算并写入长度字段。同时，消息类型必须与当前状态一致。
