---
title: Remote MCP Server
layer: 应用工程层
module: Tool Calling / Function Calling / MCP / Connectors
status: active
created: 2026-04-23
updated: 2026-04-23
tags: [knowledge-card, ai]
---

# Remote MCP Server

> 导航：[[知识卡/应用工程层/0_应用工程层|应用工程层]] · [[知识卡/应用工程层/Tool Calling_Function Calling_MCP_Connectors/0_Tool Calling_Function Calling_MCP_Connectors|Tool Calling / Function Calling / MCP / Connectors]] · [[知识图谱/AI知识地图|AI知识地图]]

## 定义
Remote MCP Server 是 Tool Calling / Function Calling / MCP / Connectors 中的一项核心能力或机制，用于组织和发挥 工具接入、调用闭环、权限与执行控制。

## 核心问题
核心问题是：Remote MCP Server 能解决什么问题、边界在哪里、应该和哪些机制组合使用。

## 关键关系
- 所属结构：[[知识卡/应用工程层/Tool Calling_Function Calling_MCP_Connectors/0_Tool Calling_Function Calling_MCP_Connectors|Tool Calling / Function Calling / MCP / Connectors]]
- 同目录知识：[[知识卡/应用工程层/Tool Calling_Function Calling_MCP_Connectors/MCP Authorization|MCP Authorization]]、[[知识卡/应用工程层/Tool Calling_Function Calling_MCP_Connectors/Connectors|Connectors]]
- 上下游模块：[[知识卡/应用工程层/Skills_Workflow_Agent/0_Skills_Workflow_Agent|Skills / Workflow / Agent]]、[[知识卡/企业落地层/安全_合规_治理/0_安全_合规_治理|安全 / 合规 / 治理]]

## 应用场景
- 让模型连接数据库、CRM、工单、日历、邮箱等外部系统。
- 控制工具调用的参数、超时、重试和幂等。
- 在企业环境下处理鉴权、审计和权限隔离。

## 常见误区
- 给模型太多工具却没有权限和失败控制。
- 忽略幂等、超时、重试和结果回传格式。

## 关联知识点
- [[知识卡/应用工程层/Tool Calling_Function Calling_MCP_Connectors/MCP Authorization|MCP Authorization]]
- [[知识卡/应用工程层/Tool Calling_Function Calling_MCP_Connectors/Connectors|Connectors]]
- [[知识卡/应用工程层/Tool Calling_Function Calling_MCP_Connectors/MCP Tools|MCP Tools]]
- [[知识卡/应用工程层/Tool Calling_Function Calling_MCP_Connectors/OAuth|OAuth]]
- [[知识卡/应用工程层/Skills_Workflow_Agent/0_Skills_Workflow_Agent|Skills / Workflow / Agent]]
- [[知识卡/企业落地层/安全_合规_治理/0_安全_合规_治理|安全 / 合规 / 治理]]
