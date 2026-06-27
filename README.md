# LangChain Agent 学习笔记

基于 LangChain 框架的 Agent 开发学习项目，边学边练、边沉淀文档。

## 学习进度

| 阶段 | 主题 | 状态 | 代码 / 文档 |
|------|------|------|-------------|
| 0 | LangChain 是什么 | ✅ 已完成 | [docs/00_langchain是什么.md](docs/00_langchain是什么.md) |
| 1 | 环境搭建 | 进行中 | [docs/01_langchain环境搭建.md](docs/01_langchain环境搭建.md) |
| 2 | LLM 基础调用 | ✅ 已完成 | [00_LLM应用.ipynb](00_LLM应用.ipynb) · [docs/02_LLM基础调用.md](docs/02_LLM基础调用.md) |
| 3 | Ollama 本地模型 | ✅ 已完成 | [docs/03_ollama搭建本地模型.md](docs/03_ollama搭建本地模型.md) |
| 4 | 提示词模板 | ✅ 已完成 | [01_langchain关键对象.ipynb](01_langchain关键对象.ipynb) · [docs/04_langchain提示词模板.md](docs/04_langchain提示词模板.md) |
| 5 | 输出格式化 | ✅ 已完成 | [01_langchain关键对象.ipynb](01_langchain关键对象.ipynb) · [docs/05_输出格式化.md](docs/05_输出格式化.md) |
| 6 | LCEL 链式调用 | 待开始 | — |
| 7 | Tools 与 Agent | 待开始 | — |
| 8 | RAG 检索增强 | 待开始 | — |

## 项目结构

```
langchain_agent/
├── README.md                 # 本文件：学习地图与快速入口
├── .env                      # API Key 配置（勿提交到 Git）
├── 00_LLM应用.ipynb              # 第一章实验：云端 + 本地 LLM 调用
├── 01_langchain关键对象.ipynb    # 第二章实验：PromptTemplate / ChatPromptTemplate
└── docs/
    ├── 00_langchain是什么.md
    ├── 01_langchain环境搭建.md
    ├── 02_LLM基础调用.md
    ├── 03_ollama搭建本地模型.md
    ├── 04_langchain提示词模板.md
    └── 05_输出格式化.md
```

## 环境要求

- **Conda 环境**：`langagent`
- **Python**：3.10.20
- **核心依赖**（截至 2026-06）：

| 包名 | 版本 |
|------|------|
| langchain-core | 1.4.8 |
| langchain-openai | 1.3.3 |
| langchain-ollama | 1.1.0 |
| langchain-deepseek | 1.1.0 |
| python-dotenv | 1.2.2 |

## 快速开始

### 1. 配置环境变量

在项目根目录创建 `.env`：

```env
DEEPSEEK_API_KEY=your_deepseek_api_key
LANGCHAIN_API_KEY=your_langsmith_api_key
LANGCHAIN_TRACING_V2=true
```

### 2. 启动 Jupyter

```bash
conda activate langagent
jupyter notebook
```

打开 `00_LLM应用.ipynb`，按顺序运行单元格。

### 3. 本地模型（可选）

需先安装 [Ollama](https://ollama.com/) 并下载模型（`pull` 或 `run` 均可，本地没有时 `run` 会自动下载）：

```bash
ollama run deepseek-r1:1.5b
```

## 第一章做了什么

`00_LLM应用.ipynb` 按序号包含三节：

| 序号 | 内容 | 文档 |
|------|------|------|
| 01 | 加载 `.env` 与 API Key | [01_langchain环境搭建.md](docs/01_langchain环境搭建.md) |
| 02 | 云端 DeepSeek（`ChatOpenAI`） | [02_LLM基础调用.md](docs/02_LLM基础调用.md) |
| 03 | 本地 Ollama（`OllamaLLM`） | [03_ollama搭建本地模型.md](docs/03_ollama搭建本地模型.md) |

## 第二章做了什么

`01_langchain关键对象.ipynb` 按序号包含三节：

| 序号 | 内容 | 文档 |
|------|------|------|
| 01 | 加载 `.env` 与 API Key | [01_langchain环境搭建.md](docs/01_langchain环境搭建.md) |
| 02 | 字符串模板（`PromptTemplate`） | [04_langchain提示词模板.md](docs/04_langchain提示词模板.md) |
| 03 | 对话模板（`ChatPromptTemplate`） | [04_langchain提示词模板.md](docs/04_langchain提示词模板.md) |
| 04 | 输出格式化（`StructuredOutputParser`） | [05_输出格式化.md](docs/05_输出格式化.md) |

## 文档撰写原则

- **问题驱动**：记录「要解决什么 → 怎么做 → 结论是什么」
- **可复现**：保留环境版本、配置项；Python 验证代码在文档中以 `python` 代码块呈现，不用 `python -c`
- **保留踩坑**：错误信息与修复方式一并记录

## 参考资料

- [LangChain 官方文档](https://python.langchain.com/)
- [LangSmith 平台](https://smith.langchain.com/)
- [DeepSeek API 文档](https://platform.deepseek.com/api-docs/)
- [Ollama 官网](https://ollama.com/)