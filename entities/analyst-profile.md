---
title: Analyst Profile
created: 2026-07-08
updated: 2026-07-08
type: entity
tags: [model, architecture]
sources: [raw/articles/mixroute-hermes-multi-agent-setup-2026-06-20.md]
confidence: high
---

# Analyst Profile

Analyst 是 [hermes-multi-agent-setup](./hermes-multi-agent-setup.md) 中的**知識綜合者**，負責把 Scout 蒐集的原始 signal 轉為結構化 wiki 筆記。

## 職責
- 讀取 inbox 中新進檔案
- 綜合多個來源，輸出結構化筆記
- 為每個主張標註信心等級：`[verified]` / `[likely]` / `[unverified]` / `[conflicting]`
- 標記與既有 wiki 內容矛盾的條目
- 將處理過的檔案移動到 processed/

## 輸出位置
`~/obsidian-wiki/`，透過 LLM Wiki skill 寫入

## 模型選擇
**需要強推理模型**。因為 Briefer 和最終使用者的輸出品質完全建立在 Analyst 的輸出上，推理品質越弱，整條管線越弱。

## 排程
通常每日一次，搭配 [wakeAgent-gate](./wakeAgent-gate.md)，只有在 inbox 非空時才喚醒。

## 可選增強：NotebookLM
透過 `notebooklm-mcp-cli` 連接 NotebookLM 作為 MCP，讓 Analyst 把整批來源一次餵進去做更深層的 cross-source 綜合。缺點是依賴 browser automation，若 Google 改動內部結構可能失效，因此 soul 中必須寫入 fallback：直接以 /goal 執行綜合。

## SOUL.md 原則
- 精確、證據導向
- 禁止呈現無根據主張為事實
- 禁止刪除 wiki 條目，只能更新或標記矛盾

## 相關頁面
- [hermes-multi-agent-setup](./hermes-multi-agent-setup.md)
- [scout-profile](./scout-profile.md)
- [briefer-profile](./briefer-profile.md)
- [wakeAgent-gate](./wakeAgent-gate.md)
- [llm-wiki](./llm-wiki.md)
