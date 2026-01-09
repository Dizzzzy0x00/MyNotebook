---
description: 自动化对抗测试框架类工具
---

# Adversarial Testing  Tools of LLM

## 自动化对抗测试框架类工具

### PyRIT（Microsoft）

PyRIT 是由微软开源的生成式 AI 红队框架，旨在系统性地发现 LLM 应用中的安全风险。该框架将对抗测试抽象为“攻击策略 + 评分器 + 执行引擎”的组合流程，支持自动化生成攻击输入并对输出进行结构化评估。

**支持的对抗类型**

* 提示注入（Prompt Injection）
* 越狱与角色扮演攻击
* 多轮对话操纵
* 策略组合攻击（Chain-of-Attacks）

**核心特性**

* 模块化攻击生成器与评分器
* 支持黑盒 API 与应用级测试
* 适用于系统级红队评估与安全审计

{% embed url="https://github.com/microsoft/PyRIT" %}

### Promptfoo（Red Team Mode）

**工具定位**\
Promptfoo 是一个面向 LLM 应用的开源测试与评测框架，其 Red Team 模块可自动生成对抗提示并对模型行为进行批量测试，广泛用于 CI/CD 流水线中的安全回归测试。

**支持的对抗类型**

* 提示注入
* 角色扮演越狱
* 逻辑一致性 / 自相矛盾检测（通过断言与评分函数）

**核心特性**

* 自动生成对抗测试样本
* 支持多模型、多提示并行评测
* 可定义一致性、拒答率、违规率等指标<br>

{% embed url="https://github.com/promptfoo/promptfoo" %}

### &#x20;DeepTeam

**工具定位**\
DeepTeam 是一个轻量级、可扩展的 LLM 红队框架，强调**快速扫描模型或 LLM 应用的潜在安全漏洞**，适合用于早期风险评估与攻击面分析。

**支持的对抗类型**

* 提示注入
* 越狱与角色扮演攻击
* 规则绕过与策略冲突

**核心特性**

* 内置多种攻击模板
* 支持自动组合攻击策略
* 适用于模型与应用级安全测试

{% embed url="https://github.com/confident-ai/deepteam" %}

### garak（NVIDIA）

**工具定位**\
garak 是由 NVIDIA 开源的 LLM 漏洞扫描器，类似于“面向语言模型的自动化漏洞扫描工具”，内置大量探针（probe）与检测器（detector），用于系统性暴露模型弱点。

**支持的对抗类型**

* 提示注入
* 误导性陈述与虚假前提
* 逻辑矛盾与强制反驳（Must-Contradict）
* 安全策略绕过

**核心特性**

* 覆盖面广的攻击探针集合
* 可复现实验型测试流程
* 对逻辑一致性与反事实场景支持较强

{% embed url="https://github.com/NVIDIA/garak" %}

## 提示注入专项工具

### promptmap2

**工具定位**\
promptmap2 是一个专注于提示注入攻击的自动化扫描工具，通过双 LLM 架构（攻击模型 + 判定模型）判断提示注入是否成功，适合用于注入漏洞挖掘与防御评测。

**支持的对抗类型**

* 提示注入
* 提示泄露
* 指令覆盖攻击

**核心特性**

* 支持黑盒与半黑盒测试
* 自动化生成注入 payload
* 支持 HTTP/API 级应用测试

{% embed url="https://github.com/utkusen/promptmap" %}

### Giskard（Open Source）

**工具定位**\
Giskard 提供了一个统一的 Python 接口，用于对 LLM 应用进行安全与鲁棒性评测，包含提示注入检测与对抗测试能力。

**支持的对抗类型**

* 提示注入
* 模型鲁棒性与异常行为检测

**核心特性**

* 易于集成到 Python 项目
* 适合评测与审计场景
* 与数据集和测试报告联动<br>

{% embed url="https://github.com/Giskard-AI/giskard" %}

### llm-attacks（Zou et al.）

**工具定位**\
llm-attacks 是一项研究型开源项目，提出了**自动生成可迁移对抗后缀（adversarial suffix）**&#x7684;方法，用于系统性诱导模型违反对齐目标。

**支持的对抗类型**

* 越狱攻击
* 对齐绕过
* 角色扮演诱导

**核心特性**

* 算法级对抗样本生成
* 具备跨模型迁移性
* 适合用于生成对抗训练数据

**开源地址**\
[https://github.com/llm-attacks/llm-attacks](https://github.com/llm-attacks/llm-attacks?utm_source=chatgpt.com)

## 开源基准与对抗数据集

### Open-Prompt-Injection

**定位**\
该项目提供了统一的提示注入攻击与防御评测框架，源自 USENIX Security 相关研究，包含规范化的攻击样本与评测流程。

**覆盖对抗类型**

* 提示注入
* 指令劫持

**开源地址**\
[https://github.com/liu00222/Open-Prompt-Injection](https://github.com/liu00222/Open-Prompt-Injection?utm_source=chatgpt.com)

### Lakera PINT Benchmark

**定位**\
PINT 是一个专门用于评测提示注入防御能力的开源基准，广泛用于比较不同检测器与防护策略。

**覆盖对抗类型**

* 提示注入

**开源地址**\
[https://github.com/lakeraai/pint-benchmark](https://github.com/lakeraai/pint-benchmark?utm_source=chatgpt.com)

***

## 总结

| 对抗类型 | 推荐工具                                 |
| ---- | ------------------------------------ |
| 提示注入 | promptmap2、Promptfoo、garak、PyRIT     |
| 角色扮演 | PyRIT、Promptfoo、DeepTeam、llm-attacks |
| 逻辑矛盾 | garak、Promptfoo（一致性断言）               |
