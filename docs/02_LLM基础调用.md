# LLM 基础调用

> 对应实验代码：[00_LLM应用.ipynb](../00_LLM应用.ipynb)
> 更新时间：2026-06

## 1. 学习目标

- 使用 LangChain 调用云端大模型（DeepSeek API）
- 从 `.env` 加载 API Key，无需手动输入
- 理解 `ChatOpenAI` 与 `OllamaLLM` 的区别
- 了解 LangSmith 追踪的基本配置

## 2. 背景与概念

### LangChain 中的两类模型接口

| 类型 | 代表类 | 输入 | 输出 | 适用场景 |
|------|--------|------|------|----------|
| **LLM** | `OllamaLLM` | 字符串 | 字符串 | 简单文本补全 |
| **ChatModel** | `ChatOpenAI` | 字符串 / 消息列表 | `AIMessage` | 多轮对话、Agent |

### 云端 vs 本地

| 方式 | 优点 | 缺点 |
|------|------|------|
| 云端 API（DeepSeek） | 能力强、速度快 | 需要 API Key、有费用 |
| 本地 Ollama | 免费、数据不出本机 | 依赖硬件、小模型能力有限（详见 [03_ollama搭建本地模型.md](03_ollama搭建本地模型.md)） |

## 3. 环境准备

### 依赖安装

```bash
conda activate langagent
pip install langchain-openai langchain-ollama python-dotenv
```

### `.env` 配置

```env
# DeepSeek 云端 API
DEEPSEEK_API_KEY=your_deepseek_api_key

# LangSmith 追踪（可选，建议学习阶段开启）
LANGCHAIN_API_KEY=your_langsmith_api_key
LANGCHAIN_TRACING_V2=true
```

> **注意**：`.env` 不要提交到 Git。`load_dotenv` 默认不会覆盖内核中已有的环境变量，建议使用 `override=True` 并在 Jupyter 中向上查找 `.env` 路径。

## 4. 实验：云端 DeepSeek 调用

### 核心思路

DeepSeek 提供 **OpenAI 兼容 API**，因此可以用 `ChatOpenAI`，只需修改 `base_url` 和 `api_key`：

```
你的代码 → ChatOpenAI → https://api.deepseek.com/v1 → DeepSeek 模型
```

### 代码

```python
import os
from pathlib import Path

from dotenv import load_dotenv
from langchain_openai import ChatOpenAI

env_path = next(
    p / ".env" for p in [Path.cwd(), *Path.cwd().parents] if (p / ".env").exists()
)
load_dotenv(env_path, override=True)

# LangSmith 追踪
os.environ["LANGCHAIN_TRACING_V2"] = os.getenv("LANGCHAIN_TRACING_V2", "true")
if os.getenv("LANGCHAIN_API_KEY"):
    os.environ["LANGCHAIN_API_KEY"] = os.getenv("LANGCHAIN_API_KEY")

deepseek_api_key = os.getenv("DEEPSEEK_API_KEY")
if not deepseek_api_key:
    raise ValueError("没有找到 DEEPSEEK_API_KEY，请先在 .env 文件中配置。")

model = ChatOpenAI(
    model="deepseek-chat",
    api_key=deepseek_api_key,
    base_url="https://api.deepseek.com/v1",
)

response = model.invoke("你好，请问你是谁？")
print(response.content)
```

### 关键点

- `api_key`：从 `.env` 的 `DEEPSEEK_API_KEY` 读取，**不是** `OPENAI_API_KEY`
- `base_url`：必须指向 DeepSeek，否则请求会发到 OpenAI 服务器
- `response.content`：`ChatModel` 返回的是 `AIMessage` 对象，文本在 `.content` 属性中

## 5. 常见错误与排查

| 报错 | 原因 | 解决方案 |
|------|------|----------|
| `AuthenticationError: Incorrect API key` | 用了 `ChatOpenAI` 但未配置 `base_url`，或 Key 是占位符 | 设置 `base_url="https://api.deepseek.com/v1"`，检查 `.env` 中的真实 Key |
| 读到 `你的_deepseek_api_key` 等占位符 | Jupyter 内核缓存了旧环境变量，`load_dotenv` 未覆盖 | 使用 `load_dotenv(override=True)`，并 Restart Kernel |
| `NameError: ChatOpenAI is not defined` | 缺少 import | 添加 `from langchain_openai import ChatOpenAI` |
| `ModuleNotFoundError: langchain_deepseek` | 未安装包 | `pip install langchain-deepseek` |
| `LangSmithError: 403 Forbidden` | LangSmith Key 无效或未配置 | 不影响模型调用；修正 Key 或设 `LANGCHAIN_TRACING_V2=false` |

## 6. LangSmith 简介

LangSmith 是 LangChain 官方的 **可观测性平台**，开启追踪后可在 [smith.langchain.com](https://smith.langchain.com) 查看每次调用的输入、输出、耗时和 token 用量。

与本章相关的配置：

```env
LANGCHAIN_TRACING_V2=true
LANGCHAIN_API_KEY=your_langsmith_api_key
```

关闭追踪：

```env
LANGCHAIN_TRACING_V2=false
```

## 7. 我的理解

- LangChain 不绑定某一家模型厂商，通过统一接口（`ChatOpenAI`、`OllamaLLM` 等）切换后端。
- **Agent 开发的第一步就是能稳定调通模型**，云端和本地各走一条路。
- `ChatModel` 比 `LLM` 更常用，后续 Agent、Memory、Tool 都基于 ChatModel 构建。
- `.env` + `override=True` 是在 Jupyter 中可靠加载配置的关键。