# Ollama 搭建本地模型

> 前置条件：已完成 [01_langchain环境搭建.md](01_langchain环境搭建.md) 中的 Python 环境与 `langchain-ollama` 安装；建议先完成 [02_LLM基础调用.md](02_LLM基础调用.md)。

## 学习目标

安装 Ollama 客户端，下载并验证本地模型，通过 LangChain 成功完成对话。

## 安装 Ollama

1. 从 [ollama.com](https://ollama.com/) 下载并安装 Windows 客户端
2. 安装后确保 Ollama 在后台运行（系统托盘有 Ollama 图标）
3. 安装完成后**重启 Cursor**（若在安装前已打开），否则集成终端可能找不到 `ollama` 命令

> Ollama 不通过 pip 安装，是独立的本地推理服务，默认监听 `http://localhost:11434`。

## 下载模型

模型只需下载到本地一次，以下两种方式任选其一：

```bash
# 方式一：显式下载（不进入对话）
ollama pull deepseek-r1:1.5b

# 方式二：直接运行（本地没有时会自动下载，并进入终端对话）
ollama run deepseek-r1:1.5b
```

| 命令 | 是否必须 | 说明 |
|------|----------|------|
| `ollama pull` | 可选 | 仅下载模型，不进入对话 |
| `ollama run` | 可选 | 本地无模型时**自动下载**；可顺便在终端测试 / 预热；LangChain 调用时**不需要**先执行 |

> LangChain 通过 HTTP API 调用时不会自动下载模型，需确保模型已通过 `pull` 或 `run` 存在于本地。

### 本机选型参考

测试环境为 Intel Iris Xe 核显 + 32GB 内存，推荐 `deepseek-r1:1.5b`（约 1.1GB）。更大模型（7B+）在 CPU 推理下会非常慢。

## .env 配置（可选）

Ollama 默认连接 `http://localhost:11434`，一般无需配置。仅当服务运行在非默认地址时，在 `.env` 中添加：

```env
# ── Ollama 本地服务（可选）──────────────────────────────
# 读取方：langchain-ollama 连接本地服务时读取
# OLLAMA_HOST=http://localhost:11434
```

| 环境变量 | 读取方 | 关联包 | 是否必填 |
|----------|--------|--------|----------|
| `OLLAMA_HOST` | `langchain-ollama` | `langchain-ollama` | 可选，默认 `localhost:11434` |

## 验证步骤

### 步骤 1：确认 Ollama 服务与模型

**PowerShell：**

```powershell
# 确认服务运行中
Invoke-WebRequest -Uri http://localhost:11434/api/tags -UseBasicParsing

# 确认模型已下载
ollama list
```

**若 Cursor 终端找不到 `ollama` 命令：**

```powershell
# 临时添加 PATH（仅当前终端有效）
$env:PATH += ";$env:LOCALAPPDATA\Programs\Ollama"

# 或直接使用完整路径
& "$env:LOCALAPPDATA\Programs\Ollama\ollama.exe" list
```

> 外部 PowerShell 能用、Cursor 终端不能用，是因为 Cursor 启动时缓存了旧 PATH，**完全重启 Cursor** 即可。

预期输出：`ollama list` 中能看到 `deepseek-r1:1.5b`。

### 步骤 2：验证 LangChain 调用

```bash
conda activate langagent
```

```python
from langchain_ollama.llms import OllamaLLM

model = OllamaLLM(model="deepseek-r1:1.5b")
response = model.invoke("hi")
print(response[:50])
```

预期输出：模型返回一段英文回复（首次调用可能较慢，Ollama 会自动加载模型）。

### 步骤 3：验证 Jupyter Notebook

打开 `00_LLM应用.ipynb`，运行 **03 Ollama 本地模型** 单元格，无报错即本地模型搭建完成。

## LangChain 接入

```python
from langchain_ollama.llms import OllamaLLM

model = OllamaLLM(model="deepseek-r1:1.5b")

# OllamaLLM 直接 invoke 时传入字符串，不是 dict
response = model.invoke("What is LangChain?")
print(response)
```

### 关于 `invoke` 入参类型

```python
# ❌ 错误：OllamaLLM 不接受 dict
model.invoke({"question": "What is LangChain?"})

# ✅ 正确：直接传字符串
model.invoke("What is LangChain?")

# dict 格式用于 Chain + Prompt 模板场景：
# chain = prompt | llm
# chain.invoke({"question": "..."})
```

调用流程：

```
你的代码 → langchain-ollama → http://localhost:11434 → Ollama 自动加载模型 → 返回结果
```

## 常见错误与排查

| 报错 | 原因 | 解决 |
|------|------|------|
| `ollama` 无法识别（仅 Cursor 终端） | Cursor 启动时 PATH 未包含 Ollama | 重启 Cursor，或临时添加 PATH / 使用完整路径 |
| Ollama 连接失败 / Connection refused | Ollama 服务未启动 | 打开 Ollama 客户端 |
| `ValueError: Invalid input type dict` | `OllamaLLM.invoke()` 传了字典 | 改为字符串：`model.invoke("你的问题")` |
| 首次调用很慢 | 模型正在加载到内存 | 正常现象，或用 `ollama run` 预热 |
| 模型不存在 | 模型未下载到本地 | 执行 `ollama pull deepseek-r1:1.5b` 或 `ollama run deepseek-r1:1.5b` |

## 我的理解

- `ollama pull` 和 `ollama run` 都能把模型下载到本地，`run` 在本地没有模型时会自动下载；LangChain 通过 HTTP API 调用，不依赖 CLI 是否在 PATH 中（但 Ollama 服务必须在运行，且模型需已下载）。
- Cursor 集成终端的 PATH 可能落后于系统，安装新软件后需重启 Cursor。
- 本地 1.5B 模型适合练手，复杂任务仍建议用云端 API。