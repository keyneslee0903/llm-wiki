---
title: Model Context Protocol (MCP)
created: 2026-07-09
updated: 2026-07-09
type: concept
tags: [model-context-protocol, ai-infrastructure, devops, platform-engineering, tool-integration]
sources: [stackgen.com-2e5fdc9a0d.md, devops.com-mcp-developer-productivity]
confidence: high
contested: false
---

# Model Context Protocol (MCP)

## 定義
MCP 是 Anthropic 在 2024 年底推出的 open standard protocol，標準化 AI 模型與外部工具/系統之間的溝通方式。

> 核心價值：不用為每個工具手寫 integration，寫一次 MCP server，任何支援 MCP 的 AI client 都能用。

## 關鍵轉變
```
Before MCP:
  開發者 copy/paste 資料到 ChatGPT 視窗 → 人工分析 → 手動操作工具

After MCP:
  開發者用自然語言：「show me failing pods and their recent logs」
  → AI agent 透過 MCP 直接查 Kubernetes + Prometheus + PagerDuty
  → 不用離開 IDE
```

## 2026 年生態現況
- **Microsoft**：開發團隊日常大量使用 MCP servers，稱其「solve real problems」
- **AWS**：推出 ECS / EKS / Serverless 專用 MCP servers
- **HashiCorp**：定位 MCP 為「trusted automation 與 AI 生態系之間的新介面層」
- **主流 IDE 支援**：VS Code Copilot、Cursor、Claude Code、Windsurf、Cline

## 對 Platform Engineering / DevSecOps 的應用

| 工具領域 | MCP Server | 具體價值 |
|---|---|---|
| IaC | Terraform MCP / StackGen MCP | Agent 讀/寫 Terraform，做合規審查 |
| CI/CD | GitHub MCP / Azure DevOps MCP | 自動 review PR、觸發 pipeline |
| Container | Kubernetes MCP | Agent 查 pod/event/log |
| 監控 | Prometheus MCP / Datadog MCP | 自然語言查 metrics |
| 事件 | PagerDuty MCP | 自動查 on-call、協調 response |
|  Secrets | Vault MCP | 有條件讀取 + audit log |
| 成本 | AWS Billing MCP | 「這週最貴的 10 個 resource」 |

## 與傳統 API Gateway 的差異

關鍵：API Gateway 管固定的 endpoint，MCP Gateway 管**動態發現的工具**。

| 維度 | API Gateway | MCP Gateway |
|---|---|---|
|管控對象|固定 API endpoint|Agent 動態選擇的工具|
|權限模型|API key / OAuth|Agent intent + policy-as-code|
|審查粒度|單次 request|跨多個 tool 的組合操作|
|風險檢測|固定欄位|Agent 行為模式異常偵測|

## 對團隊的啟示
1. 盤點工具鏈，列出哪些已有 MCP server — 這是 2026 下半年的基礎設施議題
2. MCP gateway 會成為企業 AI 治理的必修課，現在開始看方案不會太早
3. 先用 2-3 個高頻 bottleneck 做 pilot，不要一次全上

## 相關頁面
- [[mcp-gateway-enterprise]]
- [[ai-platform-engineering-2026]]
- [[claude-devops-integration]]
