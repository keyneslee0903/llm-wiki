---
title: Enterprise MCP Gateway
created: 2026-07-09
updated: 2026-07-09
type: concept
tags: [mcp-gateway, ai-security, devsecops, governance, platform-engineering]
sources: [integrate.io-best-mcp-gateways, platformcon-2026-mcp-gateway, live.paloaltonetworks.com-mcp-gateway-security]
confidence: high
contested: false
---

# Enterprise MCP Gateway

## 為什麼需要 MCP Gateway
當 AI agent 能透過 MCP 直接操作你的基礎設施時，傳統 API gateway 管不到 Agent 的**動態工具選擇**與**跨工具組合行為**。

MCP Gateway 提供：
- **意圖導向權限控制**：不只是 API key，而是檢查 agent 的操作是否符合 policy
- **跨操作審查**：agent 連續調用多個 tool 的組合是否安全
- **Audit immutable log**：所有操作留痕，符合合規要求
- **即時攔截**：destructive 操作在執行前阻擋

## 與 API Gateway 的核心差異
- API Gateway：固定 endpoint、固定 policy、單次檢查
- MCP Gateway：動態 tool discovery、意圖理解、組合行為分析

## 企業選型（2026）

| 產品 | 定位 |
|---|---|
| **TrueFoundry** | K8s-native enterprise AI gateway，提供 agent 流量治理 + observability |
| **Palo Alto Prisma AIRS 3.0** | 將現有 API security policy 延伸到 MCP 層 |
| **Mirantis** | Kubernetes 上跑多租戶 MCP 的 production-ready pattern |
| **Harness MCP v2** | Registry-based dispatch，context window 消耗僅 1.6% |

## DevSecOps 應用場景
1. 阻止 agent 誤刪 production resource
2. 防止 agent 拉取敏感 logs 外洩
3. 多個 agent 同時操作共享資源的樂觀鎖
4. 合規 audit trail

## 對平台團隊的意義
MCP Gateway 是 **AI 時代的平台邊界**。在 agent 能自動操作 infra 之前，必須先建立可治理的控制點。

## 相關頁面
- [[model-context-protocol]]
- [[ai-platform-engineering-2026]]
- [[claude-devops-integration]]
