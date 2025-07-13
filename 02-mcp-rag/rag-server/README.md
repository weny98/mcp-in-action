# MCP RAG 服务器

这是一个基于 MCP (Model Context Protocol) 的 RAG (Retrieval-Augmented Generation) 服务器，支持多种 AI API 提供商。

## 功能特性

- 🔄 **动态 API 选择**: 支持 DeepSeek、OpenAI、阿里云百炼、Claude 等多种 API 提供商
- 📄 **文档索引**: 支持批量文档向量化索引
- 🔍 **语义检索**: 基于 FAISS 的高效向量检索
- 🛠️ **MCP 工具**: 提供标准化的工具接口

## 支持的 API 提供商

| 提供商     | 环境变量                | API Key 变量        | 嵌入模型               |
| ---------- | ----------------------- | ------------------- | ---------------------- |
| DeepSeek   | `API_PROVIDER=deepseek` | `DEEPSEEK_API_KEY`  | text-embedding-3-small |
| OpenAI     | `API_PROVIDER=openai`   | `OPENAI_API_KEY`    | text-embedding-3-small |
| 阿里云百炼 | `API_PROVIDER=ali`      | `DASHSCOPE_API_KEY` | text-embedding-v4      |
| Claude     | `API_PROVIDER=claude`   | `ANTHROPIC_API_KEY` | text-embedding-3-small |

## 环境配置

### 1. 创建环境变量文件

在 `rag-server` 目录下创建 `.env` 文件：

```bash
# 选择 API 提供商 (deepseek, openai, ali, claude)
API_PROVIDER=deepseek

# 对应的 API Key (根据选择的提供商配置)
DEEPSEEK_API_KEY=your_deepseek_api_key
OPENAI_API_KEY=your_openai_api_key
DASHSCOPE_API_KEY=your_ali_api_key
ANTHROPIC_API_KEY=your_claude_api_key
```

### 2. 运行时切换 API 提供商

```bash
# 使用 DeepSeek
API_PROVIDER=deepseek uv run server.py

# 使用 OpenAI
API_PROVIDER=openai uv run server.py

# 使用阿里云百炼
API_PROVIDER=ali uv run server.py

# 使用 Claude
API_PROVIDER=claude uv run server.py
```

## 启动服务器

### 方法一：使用动态 API 选择（推荐）

```bash
cd mcp-in-action/02-mcp-rag/rag-server

# 使用默认配置启动
uv run server.py

# 或指定 API 提供商
API_PROVIDER=deepseek uv run server.py
```

### 方法二：使用专用服务器文件

```bash
# 使用阿里云百炼专用服务器
uv run server-ali.py
```

## 启动客户端

### 使用动态服务器

```bash
cd mcp-in-action/02-mcp-rag/rag-client

# 使用各种客户端连接动态服务器
uv run client-v1.py ../rag-server/server.py
uv run client-v2-claude.py ../rag-server/server.py
uv run client-v3-deepseek.py ../rag-server/server.py
uv run client-v4-toolcalls-deepseek.py ../rag-server/server.py
uv run client-v4-toolcalls-openai.py ../rag-server/server.py
```

### 使用专用服务器

```bash
# 连接阿里云专用服务器
uv run client-v3-deepseek-ali.py ../rag-server/server-ali.py
```

## 使用示例

启动后，系统会：

1. **初始化 RAG 系统**
2. **索引医学文档**（示例文档）
3. **提供交互式查询界面**

示例输出：

```
>>> 开始初始化 RAG 系统
使用 API 提供商: deepseek
可用工具： ['index_docs', 'retrieve_docs']
>>> 系统连接成功
>>> 正在索引医学文档...
>>> 文档索引完成

请输入您要查询的医学问题（输入'退出'结束查询）：
> 高血压怎么办？
```

## 工具说明

### `index_docs`

- **功能**: 将文档批量加入向量索引
- **参数**: `docs` (文本列表)
- **返回**: 索引状态信息

### `retrieve_docs`

- **功能**: 检索最相关的文档片段
- **参数**:
  - `query` (查询文本)
  - `top_k` (返回文档数量，默认 3)
- **返回**: 相关文档列表

## 技术架构

- **向量数据库**: FAISS (内存版)
- **嵌入模型**: 根据 API 提供商自动选择
- **协议**: MCP (Model Context Protocol)
- **传输**: stdio 或 TCP

## 故障排除

### 常见问题

1. **API 余额不足**

   ```
   Error code: 402 - Insufficient Balance
   ```

   解决：检查对应 API 提供商的账户余额

2. **API Key 错误**

   ```
   Error code: 401 - Unauthorized
   ```

   解决：检查 `.env` 文件中的 API Key 是否正确

3. **不支持的 API 提供商**
   ```
   ValueError: 不支持的 API 提供商: xxx
   ```
   解决：检查 `API_PROVIDER` 环境变量值

### 调试模式

启动时添加调试信息：

```bash
DEBUG=1 API_PROVIDER=deepseek uv run server.py
```

## 扩展开发

### 添加新的 API 提供商

在 `server.py` 的 `get_api_client()` 函数中添加：

```python
elif api_provider == "new_provider":
    return OpenAI(
        api_key=os.getenv("NEW_PROVIDER_API_KEY"),
        base_url="https://api.newprovider.com/v1"
    ), "text-embedding-model"
```

### 自定义嵌入模型

修改 `get_api_client()` 函数中的模型名称：

```python
# 例如使用不同的嵌入模型
return client, "your-custom-embedding-model"
```

## 许可证

MIT License
