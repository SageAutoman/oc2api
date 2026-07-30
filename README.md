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

你可以反复 Fork 并部署，创建多个出口 IP 不同的项目，然后在 CPA、SUB2API 等工具中配置多个域名实现轮询。

## API

兼容 OpenAI API 格式，路径均支持带 `/v1` 前缀或不带：

| 路径                                           | 方法   | 说明                            |
|----------------------------------------------|------|-------------------------------|
| `/v1/chat/completions` 或 `/chat/completions` | POST | Chat 补全（支持 `stream: true` 流式） |
| `/v1/models` 或 `/models`                     | GET  | 模型列表                          |
| `/health`                                    | GET  | 健康检查                          |
| `/ip`                                        | GET  | 查询出口 IP                       |

携带 API Key（如已配置）：

```
Authorization: Bearer <api-key>
```

## 关于 Codex 的配置

自定义模型时，Codex 无法识别其上下文窗口大小，默认值可能为 128K。可通过以下两种方式之一解决：

若选择 `deepseek-v4-flash-free` 等支持长上下文的模型，可在 Codex 中配置以下参数充分利用 1M 上下文窗口：

- **方式一**：在 CPA、SUB2API 等工具中将自定义模型映射成 GPT-5.5 等官方支持的模型名称，便于工具识别上下文。
- **方式二**：在 Codex 中手动配置以下参数：

  ```
  model_context_window = 1000000
  model_auto_compact_token_limit = 900000
  ```

    - `model_context_window`：模型的最大上下文窗口大小（token），此处设为 1,000,000，即 1M 上下文。
    - `model_auto_compact_token_limit`：当上下文占用超过此阈值时，Codex 会自动触发智能压缩以释放空间，建议设为 900,000（约窗口的
      90%）。

配置完成后，发任意消息（如 `hello`）验证能否正常回复，再用 `/status` 查看 `Model provider`、`Context window`、`Token usage`
是否生效。
