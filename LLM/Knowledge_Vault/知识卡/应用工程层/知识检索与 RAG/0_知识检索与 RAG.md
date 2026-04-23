---
title: 知识检索与 RAG
type: moc
status: active
created: 2026-04-23
updated: 2026-04-23
tags: [moc, knowledge-card, ai]
---

# 知识检索与 RAG

> 导航：[[知识卡/应用工程层/0_应用工程层|应用工程层]] · [[知识图谱/AI知识地图|AI知识地图]]

## 目录说明
这个目录聚焦 知识召回、排序、证据注入与知识更新，用于把同一模块下的知识点组织成一组可导航的知识卡。

## 内部结构
- [[知识卡/应用工程层/知识检索与 RAG/RAG|RAG]]
- [[知识卡/应用工程层/知识检索与 RAG/Query|Query]]
- [[知识卡/应用工程层/知识检索与 RAG/Query Rewrite|Query Rewrite]]
- [[知识卡/应用工程层/知识检索与 RAG/Query Expansion|Query Expansion]]
- [[知识卡/应用工程层/知识检索与 RAG/Chunking|Chunking]]
- [[知识卡/应用工程层/知识检索与 RAG/Chunk Size|Chunk Size]]
- [[知识卡/应用工程层/知识检索与 RAG/Overlap|Overlap]]
- [[知识卡/应用工程层/知识检索与 RAG/语义切片|语义切片]]
- [[知识卡/应用工程层/知识检索与 RAG/结构化切片|结构化切片]]
- [[知识卡/应用工程层/知识检索与 RAG/Embedding|Embedding]]
- [[知识卡/应用工程层/知识检索与 RAG/Embedding Model|Embedding Model]]
- [[知识卡/应用工程层/知识检索与 RAG/向量化|向量化]]
- [[知识卡/应用工程层/知识检索与 RAG/向量检索|向量检索]]
- [[知识卡/应用工程层/知识检索与 RAG/关键词检索|关键词检索]]
- [[知识卡/应用工程层/知识检索与 RAG/BM25|BM25]]
- [[知识卡/应用工程层/知识检索与 RAG/混合检索|混合检索]]
- [[知识卡/应用工程层/知识检索与 RAG/多路召回|多路召回]]
- [[知识卡/应用工程层/知识检索与 RAG/Parent-Child Retrieval|Parent-Child Retrieval]]
- [[知识卡/应用工程层/知识检索与 RAG/Multi-Vector Retrieval|Multi-Vector Retrieval]]
- [[知识卡/应用工程层/知识检索与 RAG/Metadata|Metadata]]
- [[知识卡/应用工程层/知识检索与 RAG/Metadata Filtering|Metadata Filtering]]
- [[知识卡/应用工程层/知识检索与 RAG/Reranking|Reranking]]
- [[知识卡/应用工程层/知识检索与 RAG/Cross-Encoder|Cross-Encoder]]
- [[知识卡/应用工程层/知识检索与 RAG/Top-K|Top-K]]
- [[知识卡/应用工程层/知识检索与 RAG/Recall|Recall]]
- [[知识卡/应用工程层/知识检索与 RAG/Precision|Precision]]
- [[知识卡/应用工程层/知识检索与 RAG/MRR|MRR]]
- [[知识卡/应用工程层/知识检索与 RAG/nDCG|nDCG]]
- [[知识卡/应用工程层/知识检索与 RAG/Context Precision|Context Precision]]
- [[知识卡/应用工程层/知识检索与 RAG/Context Recall|Context Recall]]
- [[知识卡/应用工程层/知识检索与 RAG/上下文污染|上下文污染]]
- [[知识卡/应用工程层/知识检索与 RAG/无答案阈值|无答案阈值]]
- [[知识卡/应用工程层/知识检索与 RAG/拒答|拒答]]
- [[知识卡/应用工程层/知识检索与 RAG/FAQ检索|FAQ检索]]
- [[知识卡/应用工程层/知识检索与 RAG/文档问答|文档问答]]
- [[知识卡/应用工程层/知识检索与 RAG/企业知识助手|企业知识助手]]
- [[知识卡/应用工程层/知识检索与 RAG/知识更新|知识更新]]
- [[知识卡/应用工程层/知识检索与 RAG/增量更新|增量更新]]
- [[知识卡/应用工程层/知识检索与 RAG/权限感知检索|权限感知检索]]
- [[知识卡/应用工程层/知识检索与 RAG/引用|引用]]
- [[知识卡/应用工程层/知识检索与 RAG/证据链|证据链]]
- [[知识卡/应用工程层/知识检索与 RAG/可追溯性|可追溯性]]
- [[知识卡/应用工程层/知识检索与 RAG/PDF解析|PDF解析]]
- [[知识卡/应用工程层/知识检索与 RAG/OCR|OCR]]
- [[知识卡/应用工程层/知识检索与 RAG/表格解析|表格解析]]

## 关联模块
- [[知识卡/应用工程层/Context Engineering_Memory_Conversation State/0_Context Engineering_Memory_Conversation State|Context Engineering / Memory / Conversation State]]
- [[知识卡/应用工程层/Evals_Guardrails_Observability/0_Evals_Guardrails_Observability|Evals / Guardrails / Observability]]
- [[知识卡/企业落地层/企业应用_商业化/0_企业应用_商业化|企业应用 / 商业化]]
