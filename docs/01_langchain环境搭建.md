# LangChain 环境搭建

> 对应实验代码：[00_LLM应用.ipynb](../00_LLM应用.ipynb) · **01 环境搭建** 单元格

## 学习目标

完成 LangChain 开发环境搭建，确保云端 API、LangSmith 追踪均可正常加载配置。

## 环境信息

- Conda 环境：`langagent`
- Python 版本：3.10.20
- 操作系统：Windows

## 依赖安装

### 创建并激活 Conda 环境

```bash
conda create -n langagent python=3.10 -y
conda activate langagent
```

### 安装 Python 包

```bash
pip install langchain langchain-openai langchain-ollama langchain-deepseek python-dotenv jupyter
```

| 包名 | 用途 |
|------|------|
| `langchain` | LangChain 主包 |
| `langchain-openai` | 调用 OpenAI 兼容 API（DeepSeek 云端） |
| `langchain-ollama` | 调用本机 Ollama 模型（需配合 Ollama 客户端，见 [03_ollama搭建本地模型.md](03_ollama搭建本地模型.md)） |
| `langchain-deepseek` | DeepSeek 专用 ChatModel（可选） |
| `python-dotenv` | 从 `.env` 加载环境变量 |
| `jupyter` | 运行 `.ipynb` 实验笔记 |

## .env 配置

在项目根目录创建 `.env` 文件（**勿提交到 Git**）：

```env
# ── DeepSeek 云端 API ──────────────────────────────────
# 读取方：代码中 os.getenv("DEEPSEEK_API_KEY")
# 用于：langchain-openai (ChatOpenAI)、langchain-deepseek (ChatDeepSeek)
DEEPSEEK_API_KEY=your_deepseek_api_key

# ── LangSmith 追踪 ─────────────────────────────────────
# 读取方：LangChain 自动读取 os.environ（由 load_dotenv 注入）
# 用于：langchain-core 运行时追踪上报
LANGCHAIN_API_KEY=your_langsmith_api_key
LANGCHAIN_TRACING_V2=true

# ── OpenAI 官方 API（可选）──────────────────────────────
# 读取方：langchain-openai 默认读取 os.environ["OPENAI_API_KEY"]
# 用于：直接调用 OpenAI 模型时
# OPENAI_API_KEY=your_openai_api_key
```

### 环境变量与包的对应关系

| 环境变量 | 通过 `getenv` / `os.environ` 读取的位置 | 关联包 | 是否必填 |
|----------|----------------------------------------|--------|----------|
| `DEEPSEEK_API_KEY` | 业务代码 `os.getenv("DEEPSEEK_API_KEY")` | `langchain-openai`、`langchain-deepseek` | 云端调用必填 |
| `LANGCHAIN_API_KEY` | LangChain 内部自动读取 `os.environ` | `langchain-core`（LangSmith） | 开启追踪时必填 |
| `LANGCHAIN_TRACING_V2` | LangChain 内部自动读取 `os.environ` | `langchain-core`（LangSmith） | 可选，默认 `false` |
| `OPENAI_API_KEY` | `ChatOpenAI` 未显式传 `api_key` 时自动读取 | `langchain-openai` | 仅直连 OpenAI 时需要 |

### 代码中加载 .env 的推荐写法

```python
import os
from pathlib import Path
from dotenv import load_dotenv

env_path = next(
    p / ".env" for p in [Path.cwd(), *Path.cwd().parents] if (p / ".env").exists()
)
load_dotenv(env_path, override=True)
```

> `override=True` 确保 `.env` 中的值覆盖 Jupyter 内核中残留的旧环境变量。

## 验证步骤

在项目根目录激活环境后，运行以下 Python 代码块（可保存为 `.py` 文件执行，或在 Jupyter 单元格中运行）。

```bash
conda activate langagent
```

### 步骤 1：验证 Python 包安装

```python
import langchain
import langchain_openai
import dotenv

print("所有包导入成功")
```

预期输出：`所有包导入成功`

### 步骤 2：验证 .env 加载

```python
import os
from pathlib import Path

from dotenv import load_dotenv

env_path = next(
    p / ".env" for p in [Path.cwd(), *Path.cwd().parents] if (p / ".env").exists()
)
load_dotenv(env_path, override=True)

for key in ["DEEPSEEK_API_KEY", "LANGCHAIN_API_KEY", "LANGCHAIN_TRACING_V2"]:
    value = os.getenv(key)
    status = "已配置" if value and not value.startswith("your_") else "未配置或仍为占位符"
    print(f"{key}: {status}")
```

预期输出：三个变量均显示 `已配置`（占位符 `your_xxx` 会提示未配置）。

### 步骤 3：验证云端 DeepSeek 调用

```python
import os
from pathlib import Path

from dotenv import load_dotenv
from langchain_openai import ChatOpenAI

load_dotenv(
    next(p / ".env" for p in [Path.cwd(), *Path.cwd().parents] if (p / ".env").exists()),
    override=True,
)

model = ChatOpenAI(
    model="deepseek-chat",
    api_key=os.getenv("DEEPSEEK_API_KEY"),
    base_url="https://api.deepseek.com/v1",
)

response = model.invoke("你好，请问你是谁？")
print(response.content[:80])
```

预期输出：模型返回一段中文回复。

### 步骤 4：验证 Jupyter Notebook

```bash
jupyter notebook
```

打开 `00_LLM应用.ipynb`，运行 **01 环境搭建** 单元格，无报错即环境搭建完成。

若运行过程中环境变量需要重置，Restart Kernel 后再次运行。

## 常见错误与排查

| 报错 | 原因 | 解决 |
|------|------|------|
| `ModuleNotFoundError` | 包未安装或内核环境不对 | `conda activate langagent` 后重新 `pip install` |
| `没有找到 DEEPSEEK_API_KEY` | `.env` 未加载或 Key 未配置 | 检查 `.env` 路径，使用 `override=True` |
| 读到占位符 `your_key` | `.env` 中仍是模板值 | 替换为真实 API Key |
| `LangSmithError: 403` | `LANGCHAIN_API_KEY` 无效 | 修正 Key 或设 `LANGCHAIN_TRACING_V2=false` |

## 我的理解

- `.env` 是本地开发的配置中心，不同包通过 `os.getenv` / `os.environ` 读取不同变量。
- `python-dotenv` 负责把 `.env` 注入到 `os.environ`，LangChain 和各模型包再从环境中取值。
- Jupyter 中务必 `Restart Kernel` 后再验证，避免旧环境变量干扰。