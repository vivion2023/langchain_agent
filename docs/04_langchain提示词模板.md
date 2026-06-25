# LangChain 提示词模板

> 对应实验代码：[01_langchain关键对象.ipynb](../01_langchain关键对象.ipynb)
> 前置条件：已完成 [02_LLM基础调用.md](02_LLM基础调用.md) 中的 `ChatOpenAI` 调用
> 更新时间：2026-06

## 1. 学习目标

- 理解提示词模板引擎的作用
- 掌握 `PromptTemplate` 与 `ChatPromptTemplate` 的区别与用法
- 将动态变量嵌入模板，稳定调用 `ChatOpenAI`

## 2. 背景与概念

### 提示词模板引擎的作用

提示词模板引擎的主要作用是：**将用户输入或动态数据结构化嵌入预定义模板，提升模型输出准确性和一致性。**

直接拼接字符串容易出现这些问题：

| 问题 | 示例 |
|------|------|
| 格式不统一 | 每次手写提示词，措辞、结构不一致 |
| 变量遗漏 | 忘记替换某个占位符，模型收到残缺指令 |
| 难以复用 | 同一套角色设定要在多处复制粘贴 |

模板引擎把「固定结构」和「可变数据」分开：模板定义骨架，运行时填入 `role`、`style`、`question` 等变量，每次生成的提示词格式一致，模型更容易按预期回答。

### 两类模板对比

| 类型 | 代表类 | 输出 | 适用场景 |
|------|--------|------|----------|
| **字符串模板** | `PromptTemplate` | 单个字符串 | 简单补全、一次性指令 |
| **对话模板** | `ChatPromptTemplate` | 消息列表（system / human / ai） | 多轮对话、需区分角色设定与用户输入 |

```
PromptTemplate        →  "你是一个数学老师,请用通俗易懂风格回答问题:勾股定理是什么？"
ChatPromptTemplate    →  [SystemMessage(...), HumanMessage(...)]
```

> LangChain 1.x 中模板类位于 `langchain_core.prompts`，不再使用旧版 `langchain.prompts`。

## 3. 环境准备

与 [02_LLM基础调用.md](02_LLM基础调用.md) 相同。

## 4. 实验一：PromptTemplate

### 核心思路

1. 用 `{变量名}` 定义占位符
2. `PromptTemplate.from_template()` 创建模板对象
3. 填充变量后交给 `chat.invoke()`

### 代码

对应 Notebook **02 PromptTemplate** 单元格：

```python
from langchain_core.prompts import PromptTemplate

# 定义模板
template = '你是一个{role},请用{style}风格回答问题:{question}'
# 创建模板对象
prompt_template = PromptTemplate.from_template(template)

# 变量的填充（方式一：format）
filled_prompt = prompt_template.format(
    role='数学老师', style='通俗易懂', question='勾股定理是什么？'
)

# 变量的填充（方式二：invoke，推荐，与 LCEL 链式调用一致）
# filled_prompt = prompt_template.invoke({
#     'role': '数学老师', 'style': '通俗易懂', 'question': '勾股定理是什么？'
# })

# 模型响应
ai_msg = chat.invoke(filled_prompt)
print(ai_msg.content)
```

### 关键点

- `format()` 返回**字符串**，可直接传给 `chat.invoke()`
- `invoke()` 接受**字典**，返回 LangChain 可运行的提示对象，后续链接（LCEL）时更统一
- `PromptTemplate` 适合单段指令，不区分 system / user 角色

## 5. 实验二：ChatPromptTemplate

### 核心思路

对话类模型通常需要 **system**（角色设定）和 **human**（用户问题）两条消息。`ChatPromptTemplate.from_messages()` 按消息类型组装模板。

### 代码

对应 Notebook **03 ChatPromptTemplate** 单元格：

```python
from langchain_core.prompts import ChatPromptTemplate

# 定义模板
sys_template = "你是数学老师,请以{style}的风格回答问题"
user_template = "请用简单易懂的方式解释: {question}"

# 创建对话模板
prompt_template = ChatPromptTemplate.from_messages([
    ('system', sys_template),
    ('human', user_template)
])

# 填充变量（方式一：format_messages）
prompt = prompt_template.format_messages(
    style='生动有趣', question='勾股定理是什么？'
)

# 填充变量（方式二：invoke）
# prompt = prompt_template.invoke({
#     'style': '生动有趣', 'question': '勾股定理是什么？'
# })

# 获取响应
msg = chat.invoke(prompt)
print(msg.content)
```

### 关键点

- `from_messages` 中常用角色：`system`、`human`（用户）、`ai`（历史回复）
- `format_messages()` 返回**消息列表**，`ChatOpenAI` 可直接消费
- 相比 `PromptTemplate`，能更清晰地分离「角色设定」与「用户提问」，多轮对话和 Agent 开发中更常用

## 6. 验证步骤

```bash
conda activate langagent
jupyter notebook
```

打开 `01_langchain关键对象.ipynb`，按顺序运行三个单元格：

| 单元格 | 内容 | 预期 |
|--------|------|------|
| 01 环境搭建 | 加载 `.env`、初始化 `chat` | 无报错 |
| 02 PromptTemplate | 单字符串模板 + 调用 | 打印勾股定理的中文解释 |
| 03 ChatPromptTemplate | system + human 模板 + 调用 | 打印带指定风格的中文解释 |

## 7. 常见错误与排查

| 报错 | 原因 | 解决 |
|------|------|------|
| `ModuleNotFoundError: langchain.prompts` | LangChain 1.x 已移除旧路径 | 改为 `from langchain_core.prompts import ...` |
| `KeyError: 'role'` | 模板占位符与填充变量名不一致 | 检查 `{role}` 与 `format` / `invoke` 传入的键名 |
| `NameError: chat is not defined` | 未先运行环境搭建单元格 | 先运行 Cell 01 |
| `ValidationError: variable missing` | 模板中有未填充的占位符 | 确保所有 `{变量}` 都传入了值 |

## 8. 我的理解

- 提示词模板是将一段发送给大模型的文字固定下来，中间部分留空，让用户填写少量信息，以达到给予大模型结构化数据的目的。
- `{role}`、`{style}`、`{question}` 这类占位符就是「留空」的位置，填进去之后整段提示词格式始终一致。
- `PromptTemplate` 适合单段文本留空；`ChatPromptTemplate` 则把 system、human 等消息分别固定，留空后再组装成结构化对话。
