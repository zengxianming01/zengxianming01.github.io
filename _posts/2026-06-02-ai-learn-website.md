---
layout: post
title: "AI 学习资料索引"
date: 2026-06-02
category: ai-learning
tags:
  - AI 学习
  - Agent
  - LangGraph
  - Prompt Engineering
  - MCP
excerpt: "记录一组学习 AI Agent、LangGraph、Prompt Engineering、MCP 与 RAG 的资料，以及对企业 AI 工作方向的一点整理。"
article_class: ai-resource-index
confidence: high
---

这篇文章用来记录我学习 AI 过程中会反复查看的一些资料。内容主要集中在 AI Agent、LangGraph、工具调用、Prompt Engineering、MCP、RAG 和评估方法上。

| 链接 | 概述 | 备注 |
|---|---|---|
| [LangGraph Overview](https://docs.langchain.com/oss/python/langgraph/overview) | LangGraph 官方文档 | 官方文档 |
| [AI Agents in LangGraph](https://www.deeplearning.ai/short-courses/ai-agents-in-langgraph/) | DeepLearning.AI 的 LangGraph 教程 | 教程 |
| [LangGraph: Multi-Agent Workflows](https://www.youtube.com/watch?v=hvAPnpSfSGo&themeRefresh=1) | LangChain 官方 YouTube 视频 | 视频 |
| [How to call functions with chat models](https://cookbook.openai.com/examples/how_to_call_functions_with_chat_models) | OpenAI Cookbook 工具调用示例 | 示例 |
| [Tool use](https://docs.anthropic.com/en/docs/build-with-claude/tool-use) | Anthropic 官方工具使用指南 | 官方文档 |
| [Persistence & Time Travel](https://langchain-ai.github.io/langgraph/concepts/persistence/) | LangGraph 持久化与时间旅行机制 | 官方文档 |
| [LangGraph Persistence](https://www.youtube.com/watch?v=YE6A5d8kNp4) | LangGraph 持久化视频讲解 | 视频 |
| [Memory](https://langchain-ai.github.io/langgraph/concepts/memory/) | LangGraph Memory 与 Store API 概念指南 | 官方文档 |
| [Cross-thread Persistence](https://langchain-ai.github.io/langgraph/how-tos/cross-thread-persistence/) | 跨线程长期记忆实现指南 | 指南 |
| [Prompting best practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices) | Anthropic 官方提示词最佳实践 | 官方文档 |
| [Prompt Engineering](https://platform.openai.com/docs/guides/prompt-engineering) | OpenAI 官方提示词工程指南 | 官方文档 |
| [Agent Skills overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview) | Anthropic Agent Skills 概览 | 官方文档 |
| [Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) | Anthropic Skill 编写最佳实践 | 官方文档 |
| [Get started with Agent Skills in the API](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/quickstart) | Anthropic Agent Skills 快速开始 | 官方文档 |
| [Agent Skills in the SDK](https://code.claude.com/docs/en/agent-sdk/skills) | Anthropic Agent SDK 中的 Skills | 官方文档 |
| [Evaluation best practices](https://platform.openai.com/docs/guides/evaluation-best-practices) | OpenAI 官方评估最佳实践 | 官方文档 |
| [MCP Introduction](https://modelcontextprotocol.io/docs/getting-started/intro) | Model Context Protocol 官方入门文档 | 官方文档 |
| [How to Build Your First MCP Server](https://www.youtube.com/watch?v=k_l_wKz1k1c) | MCP Server 入门视频 | 视频 |
| [MCP Servers](https://github.com/modelcontextprotocol/servers) | MCP Servers 官方开源实现合集 | 开源仓库 |
| [Build a RAG agent](https://docs.langchain.com/oss/javascript/langchain/rag) | LangChain 官方 RAG Agent 构建指南 | 官方文档 |
| [Evaluate a RAG application](https://docs.langchain.com/langsmith/evaluate-rag-tutorial) | LangSmith RAG 应用评估教程 | 官方文档 |

## 企业里的 AI 工作方向

目前的企业 AI 工作，大致可以分成三类：

1. **AI 基建**：模型网关、用量统计、权限控制，让其他团队能以更低成本、更稳定的方式使用 AI。
2. **内部 AI 工具**：知识库问答、通用 workflow 平台、Skill Hub、MCP 管理服务。
3. **团队 AI 提效**：理解业务，把团队经验沉淀成 Skill，并帮助它真正进入日常开发流程。
