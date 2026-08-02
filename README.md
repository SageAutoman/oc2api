# oc2api

OpenCode API 代理，部署在 Vercel，支持 SSE 流式响应。

如需本地部署或部署到其他云平台，参见 [server/](./server/) 目录。

## 部署

1. Fork 本仓库到你的 GitHub
2. 打开 [Vercel Dashboard](https://vercel.com)，点击 **Add New > Project**
3. 选择你 Fork 的仓库，点击 **Import**
4. 在 **Environment Variables** 中添加：
    - `API_KEY` — API 密钥（留空则匿名访问）
    - `DEBUG` — 设为 `true` 开启调试日志（可选）
5. 点击 **Deploy**，等待部署完成

部署完成后会得到一个 `https://<项目名>.vercel.app` 的域名。

你可以 Fork 后部署多个 Vercel Project，以创建多个出口 IP 不同的项目，然后在 [router-for-me/CLIProxyAPI](https://github.com/router-for-me/CLIProxyAPI/blob/main/README_CN.md#%E5%8A%9F%E8%83%BD%E7%89%B9%E6%80%A7)、[Wei-Shaw/sub2api](https://github.com/Wei-Shaw/sub2api) 等工具中配置多个域名实现轮询，规避 IP 限制。

## API

兼容 OpenAI API 格式，路径均支持带 `/v1` 前缀或不带：

| 路径                                           | 方法   | 说明                            |
|----------------------------------------------|------|-------------------------------|
| `/v1/chat/completions` 或 `/chat/completions` | POST | Chat 补全（支持 `stream: true` 流式） |
| `/v1/models` 或 `/models`                     | GET  | 模型列表                          |
| `/` 或 `/health`                             | GET  | 健康检查                          |
| `/ip`                                        | GET  | 查询出口 IP                       |

携带 API Key（如已配置）：

```
Authorization: Bearer <api-key>
```

## 免费模型限制

代理仅放行免费模型（`big-pickle` 及所有以 `-free` 结尾的模型），以 `deepseek-v4-flash-free` 为例：

```json
{
  "id": "deepseek-v4-flash-free",
  "limit": {
    "context": 200000,
    "output": 128000
  }
}
```

- `context`：最大上下文窗口，**200,000** tokens
- `output`：最大单次输出长度，**128,000** tokens

以上限制数据来源于接口 [https://models.opencode.ai/api.json](https://models.opencode.ai/api.json)（`opencode` key 下对应模型的 `limit` 字段），可自行查看核实，以实际使用为准。

## 推理强度（reasoning_effort）

目前 `reasoning_effort` 仅对 DeepSeek 模型生效：DeepSeek 只接受 `high` / `max`，两者原样透传，其余值（包括未指定）会被强制为 `max`；其他模型不处理该参数，保持默认值。
