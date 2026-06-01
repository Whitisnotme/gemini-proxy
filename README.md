# Gemini to OpenAI API Proxy

一个基于 Vercel Edge Runtime（或符合标准的 Edge 运行环境）构建的高性能 Gemini API 转 OpenAI 格式代理服务。支持多 Key 随机轮询、自动故障重试、Master Key 安全鉴权，并完美适配流式（Stream）与非流式响应。

## 🌟 特性

- **OpenAI 标准适配**：完美将 OpenAI 格式的 `/v1/chat/completions` 和 `/v1/models` 路由桥接到 Gemini `v1beta` 原生接口。
- **多 Key 轮询与高可用**：支持配置多个 Gemini API Key。每次请求随机选择，若遇到 `429`（限流）或 `50x`（服务错误）会自动剔除并尝试密钥池中的下一个 Key，直到全部耗尽，最大化保障可用性。
- **安全鉴权 (Master Key)**：支持自定义主密钥鉴权，内部采用密码学安全的 `SHA-256` 异步哈希和防时序攻击验证（Timing-Safe Equal）。
- **流式响应优化**：原生支持高效的 SSE（Server-Sent Events）流式分块传输，采用 `TextDecoder` 精准解码网络字节流，并自动剥离上游服务前缀，防解析崩溃。
- **完整模型列表**：针对 Gemini 官方模型列表接口设置了 `pageSize=1000` 参数，彻底解决默认仅返回 50 个模型的分页截断问题，一次性完整透传所有可用模型（包括最新变体）。
- **边缘计算架构**：基于轻量级 Edge Runtime 架构设计，无冷启动延迟，全球低时延响应。

## ⚙️ 环境变量配置

在部署时，请配置以下环境变量：

| 变量名 | 是否必填 | 示例值 | 说明 |
| :--- | :--- | :--- | :--- |
| `API_KEYS` | **必填** | `AIzaSyA...,AIzaSyB...,AIzaSyC...` | 英文逗号分隔的 Gemini API Key 列表，用于上游轮询和负载均衡。 |
| `MASTER_KEY` | 可选 | `sk-my-secure-proxy-key` | 客户端调用此代理时需要在 `Authorization: Bearer <KEY>` 中携带的密钥。若未设置，则默认使用 `API_KEYS` 中的第一个 Key 作为主密钥。 |

## 🚀 部署指南

### 部署到 Vercel

[![Deploy with Vercel](https://vercel.com/button)](https://vercel.com/new/clone?repository-url=https://github.com/Whitisnotme/gemini-proxy)

本项目完全兼容 Vercel Edge Functions。建议的项目目录结构如下：

```text
├── api
│   └── index.js       # 代理核心代码
├── vercel.json        # 路由重写配置文件
└── package.json       # 项目元数据
