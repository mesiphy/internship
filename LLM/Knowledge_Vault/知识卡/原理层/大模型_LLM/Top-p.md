---
title: Top-p
layer: 原理层
module: 大模型 / LLM
status: active
created: 2026-04-23
updated: 2026-04-23
tags: [knowledge-card, ai]
---

# Top-p

> 导航：[[知识卡/原理层/0_原理层|原理层]] · [[知识卡/原理层/大模型_LLM/0_大模型_LLM|大模型 / LLM]] · [[知识图谱/AI知识地图|AI知识地图]]

## 定义
Top-p 是 大模型 / LLM 中的关键知识点，用于理解和组织 生成式模型的输入、输出、能力控制与成本权衡。

## 核心问题
核心问题是：Top-p 在 大模型 / LLM 里承担什么作用、与哪些上下游环节相连、如何判断它是否发挥作用。

## 关键关系
- 所属结构：[[知识卡/原理层/大模型_LLM/0_大模型_LLM|大模型 / LLM]]
- 同目录知识：[[知识卡/原理层/大模型_LLM/Temperature|Temperature]]、[[知识卡/原理层/大模型_LLM/Max tokens|Max tokens]]
- 上下游模块：[[知识卡/应用工程层/Prompt 与 Structured Output/0_Prompt 与 Structured Output|Prompt 与 Structured Output]]、[[知识卡/应用工程层/Context Engineering_Memory_Conversation State/0_Context Engineering_Memory_Conversation State|Context Engineering / Memory / Conversation State]]

## 应用场景
- 设计基于 LLM 的问答、生成和助手类功能。
- 控制输出格式、成本、延迟和推理能力。
- 判断该靠 prompt、架构还是模型切换来优化效果。

## 常见误区
- 把 prompt 调整当成所有问题的万能解。
- 忽略上下文、工具、成本和延迟之间的联动。

## 关联知识点
- [[知识卡/原理层/大模型_LLM/Temperature|Temperature]]
- [[知识卡/原理层/大模型_LLM/Max tokens|Max tokens]]
- [[知识卡/原理层/大模型_LLM/User Prompt|User Prompt]]
- [[知识卡/原理层/大模型_LLM/幻觉|幻觉]]
- [[知识卡/应用工程层/Prompt 与 Structured Output/0_Prompt 与 Structured Output|Prompt 与 Structured Output]]
- [[知识卡/应用工程层/Context Engineering_Memory_Conversation State/0_Context Engineering_Memory_Conversation State|Context Engineering / Memory / Conversation State]]
