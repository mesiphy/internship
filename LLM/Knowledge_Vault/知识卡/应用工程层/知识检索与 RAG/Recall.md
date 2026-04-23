---
title: Recall
layer: 应用工程层
module: 知识检索与 RAG
status: active
created: 2026-04-23
updated: 2026-04-23
tags: [knowledge-card, ai]
---

# Recall

> 导航：[[知识卡/应用工程层/0_应用工程层|应用工程层]] · [[知识卡/应用工程层/知识检索与 RAG/0_知识检索与 RAG|知识检索与 RAG]] · [[知识图谱/AI知识地图|AI知识地图]]

## 定义
Recall 是 知识检索与 RAG 中用于判断效果、质量或系统状态的关键指标，关注的是 知识召回、排序、证据注入与知识更新。

## 核心问题
核心问题是：Recall 到底在衡量什么、阈值如何设定、异常时说明哪一环出了问题。

## 关键关系
- 所属结构：[[知识卡/应用工程层/知识检索与 RAG/0_知识检索与 RAG|知识检索与 RAG]]
- 同目录知识：[[知识卡/应用工程层/知识检索与 RAG/Top-K|Top-K]]、[[知识卡/应用工程层/知识检索与 RAG/Precision|Precision]]
- 上下游模块：[[知识卡/应用工程层/Context Engineering_Memory_Conversation State/0_Context Engineering_Memory_Conversation State|Context Engineering / Memory / Conversation State]]、[[知识卡/应用工程层/Evals_Guardrails_Observability/0_Evals_Guardrails_Observability|Evals / Guardrails / Observability]]

## 应用场景
- 构建企业知识问答、FAQ 检索和知识助手。
- 设计召回、重排、引用和权限感知检索。
- 处理知识更新、拒答与证据链展示问题。

## 常见误区
- 只关注向量检索，不看切片、重排和拒答。
- 把召回到的内容默认当成可靠证据。

## 关联知识点
- [[知识卡/应用工程层/知识检索与 RAG/Top-K|Top-K]]
- [[知识卡/应用工程层/知识检索与 RAG/Precision|Precision]]
- [[知识卡/应用工程层/知识检索与 RAG/Cross-Encoder|Cross-Encoder]]
- [[知识卡/应用工程层/知识检索与 RAG/MRR|MRR]]
- [[知识卡/应用工程层/Context Engineering_Memory_Conversation State/0_Context Engineering_Memory_Conversation State|Context Engineering / Memory / Conversation State]]
- [[知识卡/应用工程层/Evals_Guardrails_Observability/0_Evals_Guardrails_Observability|Evals / Guardrails / Observability]]
