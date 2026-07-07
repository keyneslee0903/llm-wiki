---
title: Scout Profile
created: 2026-07-08
updated: 2026-07-08
type: entity
tags: [model, architecture]
sources: [raw/articles/mixroute-hermes-multi-agent-setup-2026-06-20.md]
confidence: high
---

# Scout Profile

Scout 是 [hermes-multi-agent-setup](./concepts/hermes-multi-agent-setup.md) 中的**信號蒐集者**，負責定期搜尋來源並把發現寫入 inbox。

## 職責
- 搜尋來源（網頁、arXiv、RSS）
- 將發現存為 markdown 檔案
- **不分析、不摘要、不判斷輕重**

## 輸出位置
`~/research/inbox/`

## 模型選擇
便宜、快速的模型即可。若監控 X/Twitter，需搭配 `xurl` skill 或 Grok 模型（內建 X 搜尋）。

## 排程
建議每 3 小時執行一次，因為蒐集成本低而希望 signal 新鮮。

## SOUL.md 原則
- 極簡語調
- 每個檔案只放 source URL + 原始內容
- 禁止寫入其他 profile 的資料夾

## 相關頁面
Scout 是 [hermes-multi-agent-setup](./concepts/hermes-multi-agent-setup.md) 中的
- [analyst-profile](./entities/analyst-profile.md)
- [briefer-profile](./entities/briefer-profile.md)