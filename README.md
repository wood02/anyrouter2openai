# AnyRouter2OpenAI

将 OpenAI API 协议转换为 Anthropic API 协议的代理服务，让你可以使用 OpenAI 兼容的客户端访问 Anthropic Claude 模型。

## 功能特性

- **协议转换**：将 OpenAI Chat Completions API 请求转换为 Anthropic Messages API
- **流式响应**：完整支持 SSE 流式输出
- **多模态支持**：支持图片输入（Base64 和 URL 格式）
- **连接池复用**：HTTP 客户端连接池，提升性能
- **参数透传**：支持 temperature、top_p、stop 等参数
- **错误处理**：完善的错误处理和日志记录



## 快速开始

### 环境要求

- Python 3.10+
- [uv](https://github.com/astral-sh/uv) (推荐) 或 pip

### 安装

```bash
# 克隆项目
git clone https://github.com/wood02/anyrouter2openai.git
cd anyrouter2openai

# 使用 uv 安装依赖
uv sync

# 或使用 pip
pip install -r requirements.txt
```

### 运行

```bash
# 使用 uv
uv run python main.py

# 或使用 uvicorn（支持热重载）
uv run uvicorn main:app --reload --host 0.0.0.0 --port 9999
```

服务将在 `http://localhost:9999` 启动。

### 环境变量

| 变量名 | 默认值 | 说明 |
|--------|--------|------|
| `ANYROUTER_BASE_URL` | `https://anyrouter.top` | 上游 Anthropic API 地址 |

## API 接口

### Chat Completions

```
POST /v1/chat/completions
```

兼容 OpenAI Chat Completions API，支持流式和非流式响应。

**请求示例：**

```bash
curl http://localhost:9999/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "model": "claude-sonnet-4-20250514",
    "messages": [
      {"role": "user", "content": "Hello!"}
    ],
    "stream": true
  }'
```

**支持的参数：**

| 参数 | 类型 | 说明 |
|------|------|------|
| `model` | string | 模型名称 |
| `messages` | array | 消息列表 |
| `stream` | boolean | 是否流式输出 |
| `max_tokens` | integer | 最大输出 token 数 |
| `temperature` | float | 温度参数 |
| `top_p` | float | Top-p 采样 |
| `stop` | array | 停止序列 |

### 模型列表

```
GET /v1/models
```

获取可用模型列表。

### 健康检查

```
GET /health
```

返回服务健康状态。

## 使用示例

### Python (OpenAI SDK)

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:9999/v1",
    api_key="your-api-key"
)

# 非流式
response = client.chat.completions.create(
    model="claude-sonnet-4-20250514",
    messages=[{"role": "user", "content": "Hello!"}]
)
print(response.choices[0].message.content)

# 流式
stream = client.chat.completions.create(
    model="claude-sonnet-4-20250514",
    messages=[{"role": "user", "content": "Hello!"}],
    stream=True
)
for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
```

### 多模态（图片输入）

```python
response = client.chat.completions.create(
    model="claude-sonnet-4-20250514",
    messages=[
        {
            "role": "user",
            "content": [
                {"type": "text", "text": "这张图片里有什么？"},
                {
                    "type": "image_url",
                    "image_url": {"url": "data:image/png;base64,iVBORw0KGgo..."}
                }
            ]
        }
    ]
)
```

### cURL

```bash
# 流式请求
curl http://localhost:9999/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "model": "claude-sonnet-4-20250514",
    "messages": [{"role": "user", "content": "写一首诗"}],
    "stream": true
  }'
```

## Docker 部署

```dockerfile
FROM python:3.13-slim

WORKDIR /app
COPY . .

RUN pip install --no-cache-dir fastapi httpx uvicorn

EXPOSE 9999

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "9999"]
```

构建并运行：

```bash
docker build -t anyrouter2openai .
docker run -p 9999:9999 -e ANYROUTER_BASE_URL=https://anyrouter.top anyrouter2openai
```

## 开发

```bash
# 安装开发依赖
uv sync

# 运行开发服务器（热重载）
uv run uvicorn main:app --reload --port 9999

# 代码格式化
uv run ruff format .

# 代码检查
uv run ruff check .
```

## 协议映射

| OpenAI | Anthropic |
|--------|-----------|
| `messages[].role: system` | `system[]` |
| `messages[].role: user` | `messages[].role: user` |
| `messages[].role: assistant` | `messages[].role: assistant` |
| `max_tokens` | `max_tokens` |
| `temperature` | `temperature` |
| `top_p` | `top_p` |
| `stop` | `stop_sequences` |
| `stream: true` | `stream: true` |

## 已知问题

### Linux SSL 握手失败

在 Linux 系统上可能会遇到 SSL 握手失败的问题：

```
SSLV3_ALERT_HANDSHAKE_FAILURE
```

**测试状态：**

| 平台 | 状态 |
|------|------|
| macOS | ✅ 正常 |
| Windows | ✅ 正常 |
| Linux | ❌ SSL 握手失败 |


## License

MIT License

## 贡献

欢迎提交 Issue 和 Pull Request！