# RAG 智能客服系统

基于 LangChain + Streamlit + Chroma 构建的检索增强生成（RAG）智能客服系统，初始知识库预置了穿衣搭配相关内容，支持用户自行扩充知识库。

---

## 项目简介

本项目实现了一套完整的 RAG（Retrieval-Augmented Generation）流程，分为**离线处理**和**在线处理**两个部分：

- **离线处理（知识库更新）**：用户通过网页上传 TXT 文件，系统对文件内容进行 MD5 校验去重后，将文本切分并向量化存入 Chroma 向量数据库，从而持续扩充知识库。知识库初始内置了穿衣搭配相关知识。

- **在线处理（智能问答）**：用户在聊天界面提问，系统先将问题与向量库进行相似度匹配，检索出最相关的知识片段，再携带知识库内容和对话历史记录一并请求大语言模型（LLM），最终在网页上以流式方式展示完整的聊天记录，实现基础的 RAG 问答功能。

---

## 功能特性

- 📄 支持 TXT 文件上传，自动入库
- 🔑 基于 MD5 的内容去重，避免重复向量化
- 🔍 向量相似度检索，精准召回相关知识
- 🤖 结合知识库与对话历史的多轮对话
- ⚡ 流式输出，实时展示 AI 回复
- 🌐 简洁易用的 Streamlit Web 界面

---

## 技术栈

| 类别 | 技术 |
|------|------|
| RAG 框架 | [LangChain](https://www.langchain.com/) |
| 前端界面 | [Streamlit](https://streamlit.io/) |
| 向量数据库 | [Chroma](https://www.trychroma.com/) |
| Embedding 模型 | 阿里云 DashScope `text-embedding-v4` |
| 大语言模型 | 阿里云通义千问 `qwen3-max`（通过 DashScope API） |
| 文本分割 | LangChain `RecursiveCharacterTextSplitter` |
| 对话历史管理 | LangChain `RunnableWithMessageHistory` |

---

## 项目结构

```
rag_test/
├── app_qa.py               # 在线问答前端（Streamlit）
├── app_file_uploader.py    # 知识库更新前端（Streamlit）
├── rag.py                  # RAG 核心链路（检索 + 提示词 + LLM）
├── knowledge_base.py       # 知识库服务（文本切分、向量化、MD5 去重）
├── vector_stores.py        # 向量库封装（Chroma 检索器）
├── file_history_store.py   # 对话历史持久化
├── config_data.py          # 全局配置（模型名称、分块参数等）
├── chroma_db/              # Chroma 向量数据库本地存储
├── chat_history/           # 对话历史本地存储
├── data/                   # 原始知识库文件
└── md5.text                # 已入库内容的 MD5 记录（去重用）
```

---

## 快速开始

### 1. 安装依赖

```bash
pip install langchain langchain-community langchain-chroma streamlit dashscope
```

### 2. 配置 API Key

在运行前，设置阿里云 DashScope 的 API Key 环境变量：

```bash
export DASHSCOPE_API_KEY="your_api_key_here"
```

### 3. 启动知识库更新服务（离线）

```bash
streamlit run app_file_uploader.py
```

访问后上传 TXT 文件，内容将自动向量化并存入知识库。

### 4. 启动智能问答服务（在线）

```bash
streamlit run app_qa.py
```

访问聊天界面，即可开始基于知识库的多轮问答。

---

## 工作流程

```
用户上传 TXT 文件
       │
       ▼
  MD5 去重检查 ──── 已存在 ──→ 跳过
       │
     新内容
       │
       ▼
  文本切分（RecursiveCharacterTextSplitter）
       │
       ▼
  Embedding 向量化（text-embedding-v4）
       │
       ▼
  存入 Chroma 向量数据库


用户提问
       │
       ▼
  向量相似度检索（Chroma Retriever）
       │
       ▼
  构造 Prompt（知识片段 + 对话历史 + 问题）
       │
       ▼
  调用 LLM（qwen3-max，流式输出）
       │
       ▼
  Streamlit 实时展示回复
```
