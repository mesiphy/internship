---
title: Trace
layer: 应用工程层
module: Evals / Guardrails / Observability
group: Observability
status: active
created: 2026-04-23
updated: 2026-04-23
tags: [knowledge-card, ai]
---

# Trace

> 导航：[[知识卡/应用工程层/0_应用工程层|应用工程层]] · [[知识卡/应用工程层/Evals_Guardrails_Observability/0_Evals_Guardrails_Observability|Evals / Guardrails / Observability]] · [[知识卡/应用工程层/Evals_Guardrails_Observability/Observability/0_Observability|Observability]] · [[知识图谱/AI知识地图|AI知识地图]]

## 定义
Trace 是 Evals / Guardrails / Observability 中的观测与定位抓手，用于连接运行现象、版本变化和问题归因。

## 核心问题
核心问题是：如何借助 Trace 观察系统运行状态，并把现象准确归因到模型、检索、工具或流程。

## 关键关系
- 所属结构：[[知识卡/应用工程层/Evals_Guardrails_Observability/0_Evals_Guardrails_Observability|Evals / Guardrails / Observability]] · [[知识卡/应用工程层/Evals_Guardrails_Observability/Observability/0_Observability|Observability]]
- 同目录知识：[[知识卡/应用工程层/Evals_Guardrails_Observability/Observability/日志|日志]]、[[知识卡/应用工程层/Evals_Guardrails_Observability/Observability/Span|Span]]
- 上下游模块：[[知识卡/应用工程层/模型优化与定制化/0_模型优化与定制化|模型优化与定制化]]、[[知识卡/企业落地层/安全_合规_治理/0_安全_合规_治理|安全 / 合规 / 治理]]

## 应用场景
- 追踪多轮任务、工具调用和检索链路。
- 监控延迟、错误、token 消耗和成本异常。
- 把线上问题回溯到具体版本和步骤。

## 常见误区
- 有日志但没有统一追踪和归因口径。
- 能看到错误，却找不到对应的 prompt、模型或工具版本。

## 关联知识点
- [[知识卡/应用工程层/Evals_Guardrails_Observability/Observability/日志|日志]]
- [[知识卡/应用工程层/Evals_Guardrails_Observability/Observability/Span|Span]]
- [[知识卡/应用工程层/Evals_Guardrails_Observability/Observability/Tool Log|Tool Log]]
- [[知识卡/应用工程层/Evals_Guardrails_Observability/Observability/Retrieval Log|Retrieval Log]]
- [[知识卡/应用工程层/模型优化与定制化/0_模型优化与定制化|模型优化与定制化]]
- [[知识卡/企业落地层/安全_合规_治理/0_安全_合规_治理|安全 / 合规 / 治理]]
