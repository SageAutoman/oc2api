# oc2api

纯 Go 实现的 OpenCode API 代理服务，支持 SSE 流式响应。

## 配置

编辑 `config.yaml`：

| 配置项          | 默认值            | 说明               |
|--------------|----------------|------------------|
| `port`       | `8080`         | 监听端口             |
| `api-key`    | 空              | API 密钥（不设置则匿名访问） |
| `debug`      | `false`        | 调试日志             |
| `timeout-ms` | `300000` (5分钟) | 上游请求超时时间         |

## 本地部署

```bash
go build -ldflags="-s -w" -trimpath -o main main.go
./main
```

## Docker 部署

```bash
docker compose up -d
```

## Serverless 部署

本地/Docker 部署出口 IP 固定，建议部署到阿里云函数计算、腾讯云函数 等 Serverless 环境实现多出口 IP 轮询，规避速率限制。

```bash
GOOS=linux GOARCH=amd64 go build -ldflags="-s -w" -trimpath -o main main.go
# 将 main + config.yaml 上传至云函数代码空间
# chmod +x ./main
# 启动命令: ./main
# 监听端口: 8080
```

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
