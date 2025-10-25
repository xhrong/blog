---
title: 《Agent设计模式》内容摘要 
tags: [AI，Agent]
grammar_cjkRuby: true
categories: [AI]
date: 2025-10-25
---

## 概述
AI 范式发生了巨大转变，从简单自动化迈向复杂自主系统（见图 2）。最初，工作流依赖基础提示和触发器，利用 LLM 处理数据。随后，检索增强生成（RAG）技术出现，通过事实信息提升模型可靠性。接着，单体智能体诞生，能够调用多种工具。如今，我们正步入智能体 AI 时代，多个专业智能体协作完成复杂目标，AI 的协同能力实现了质的飞跃.
![alt text](./images/image.png)
## Agent设计模式

### 提示链（Prompt Chaining）
- 提示链将复杂任务拆解为一系列小型、聚焦步骤，亦称流水线模式。
- 每步链条包含一次 LLM 调用或处理逻辑，以上一步输出为输入。
- 该模式提升了与语言模型复杂交互的可靠性和可管理性。
- LangChain/LangGraph、Google ADK 等框架为多步序列定义、管理和执行提供了强大工具。

![alt text](./images/image-1.png)

