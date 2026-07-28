# oc2api

OpenCode API 代理，部署在 Vercel，支持 SSE 流式响应。

## 部署

1. Fork 本仓库到你的 GitHub

2. 打开 [Vercel Dashboard](https://vercel.com)，点击 **Add New > Project**

3. 选择你 Fork 的仓库，点击 **Import**

4. 在 **Environment Variables** 中添加：
   - `API_KEY` — API 密钥（留空则匿名访问）
   - `DEBUG` — 设为 `true` 开启调试日志（可选）

5. 点击 **Deploy**，等待部署完成

部署完成后会得到一个 `https://<项目名>.vercel.app` 的域名。

你可以反复 Fork 并部署，创建多个出口 IP 不同的项目，然后在 CPA、SUB2API 等工具中配置多个域名实现轮询。

## API

### 健康检查

```bash
curl https://<你的域名>/
```

返回：

```json
{
  "status": "ok",
  "version": "v1.0.0",
  "endpoints": [
    "/v1/chat/completions",
    "/chat/completions",
    "/v1/models",
    "/models",
    "/ip"
  ]
}
```

表示部署成功。

### 聊天补全（流式）

```bash
curl -N https://<你的域名>/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <api-key>" \
  -d '{
    "model": "<model-name>",
    "messages": [{"role": "user", "content": "Hello"}],
    "stream": true
  }'
```

也支持 `POST /chat/completions`（不带 `/v1` 前缀）。

### 获取模型列表

```bash
curl https://<你的域名>/v1/models
```

也支持不带 `/v1` 前缀的路径：`/models`。

### 查看出口 IP

```bash
curl https://<你的域名>/ip
```

## 关于 Codex 的配置

若选择 `deepseek-v4-flash-free` 等支持长上下文的模型，可在 Codex 中配置以下参数充分利用 1M 上下文窗口：

```
model_auto_compact_token_limit = 900000
model_context_window = 1000000
```

- `model_context_window`：模型的最大上下文窗口大小（token），此处设为 1,000,000，即 1M 上下文。
- `model_auto_compact_token_limit`：当上下文占用超过此阈值时，Codex 会自动触发智能压缩以释放空间，建议设为 900,000（约窗口的 90%）。

配置完成后，发任意消息（如 `hello`）验证能否正常回复，再用 `/status` 查看 `Model provider`、`Context window`、`Token usage` 是否生效。
