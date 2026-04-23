---
title: Latency
layer: 应用工程层
module: Evals / Guardrails / Observability
group: Observability
status: active
created: 2026-04-23
updated: 2026-04-23
tags: [knowledge-card, ai]
---

# Latency

> 导航：[[知识卡/应用工程层/0_应用工程层|应用工程层]] · [[知识卡/应用工程层/Evals_Guardrails_Observability/0_Evals_Guardrails_Observability|Evals / Guardrails / Observability]] · [[知识卡/应用工程层/Evals_Guardrails_Observability/Observability/0_Observability|Observability]] · [[知识图谱/AI知识地图|AI知识地图]]

## 定义
Latency 是 Evals / Guardrails / Observability 中用于判断效果、质量或系统状态的关键指标，关注的是 日志、追踪、成本与问题归因。

## 核心问题
核心问题是：Latency 到底在衡量什么、阈值如何设定、异常时说明哪一环出了问题。

## 关键关系
- 所属结构：[[知识卡/应用工程层/Evals_Guardrails_Observability/0_Evals_Guardrails_Observability|Evals / Guardrails / Observability]] · [[知识卡/应用工程层/Evals_Guardrails_Observability/Observability/0_Observability|Observability]]
- 同目录知识：[[知识卡/应用工程层/Evals_Guardrails_Observability/Observability/Token Usage|Token Usage]]、[[知识卡/应用工程层/Evals_Guardrails_Observability/Observability/Error Taxonomy|Error Taxonomy]]
- 上下游模块：[[知识卡/应用工程层/模型优化与定制化/0_模型优化与定制化|模型优化与定制化]]、[[知识卡/企业落地层/安全_合规_治理/0_安全_合规_治理|安全 / 合规 / 治理]]

## 应用场景
- 追踪多轮任务、工具调用和检索链路。
- 监控延迟、错误、token 消耗和成本异常。
- 把线上问题回溯到具体版本和步骤。

## 常见误区
- 有日志但没有统一追踪和归因口径。
- 能看到错误，却找不到对应的 prompt、模型或工具版本。

## 关联知识点
- [[知识卡/应用工程层/Evals_Guardrails_Observability/Observability/Token Usage|Token Usage]]
- [[知识卡/应用工程层/Evals_Guardrails_Observability/Observability/Error Taxonomy|Error Taxonomy]]
- [[知识卡/应用工程层/Evals_Guardrails_Observability/Observability/Retrieval Log|Retrieval Log]]
- [[知识卡/应用工程层/Evals_Guardrails_Observability/Observability/Prompt Version|Prompt Version]]
- [[知识卡/应用工程层/模型优化与定制化/0_模型优化与定制化|模型优化与定制化]]
- [[知识卡/企业落地层/安全_合规_治理/0_安全_合规_治理|安全 / 合规 / 治理]]
