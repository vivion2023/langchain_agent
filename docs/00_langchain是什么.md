# LangChain 是什么

## 学习目标

了解 LangChain 是什么、它解决什么问题，以及它与 Agent 开发的关系。

## 核心概念

LangChain 是一个**开源框架**，帮助开发者构建基于大语言模型（LLM）的智能应用，也就是 Agent。可以把它理解为 **Agent 开发的工具集合**——提供统一的模块和接口，把 LLM、数据、工具、记忆等能力组装起来。

## LangChain 的作用

LangChain 解决的是 LLM 的一些「短板」，让 Agent 能干更多事，比如：

1. **连接外部数据**：LLM 内置的知识是静态的，LangChain 能让它去查数据库、搜网页、调 API，获取最新信息。
2. **记东西**：帮 Agent 记住对话上下文（短期记忆）或用户偏好（长期记忆），让交互更个性化。
3. **用工具**：让 Agent 能调用各种工具，比如计算器、天气 API，甚至跑 Python 代码。
4. **复杂任务拆解**：把用户的大需求（比如「帮我计划旅行」）拆成小步骤，逐步完成。

## 与 Agent 开发的关系

LangChain 很像 Agent 的「万能工具箱」，但没那么「万能」——它提供了很多现成的模块，比如：

- **链（Chains）**：把多个操作串起来，比如先理解用户意图，再查数据，最后生成回答。
- **工具（Tools）**：让 Agent 能用外部功能，比如搜索、计算、翻译。
- **记忆（Memory）**：存对话历史或用户数据。
- **检索（RAG）**：从知识库或网上抓取最新信息。

在本学习路线中，后续章节会依次接触这些能力：先调通 LLM（[02_LLM基础调用.md](02_LLM基础调用.md)），再逐步扩展到 Prompt、Tools、Agent、RAG 等。

## 我的理解

- LLM 本身只会「说话」，LangChain 负责把它变成能**行动**的 Agent。
- 不必从零造轮子，LangChain 把常见模式（调用模型、连工具、存记忆）封装好了。
- 学习 Agent 开发，本质上是学习如何用 LangChain 把这些模块组合起来。

## 参考资料

- [LangChain 中文网](https://www.langchain.com.cn/)
- [LangChain 官方文档](https://python.langchain.com/)