---
title: Hermes Agent Multi-Agent Setup
created: 2026-07-08
updated: 2026-07-08
type: concept
tags: [model, architecture, concept, comparison]
sources: [raw/articles/mixroute-hermes-multi-agent-setup-2026-06-20.md]
confidence: high
---

# Hermes Agent Multi-Agent Setup

在多代理系統中，將工作拆分給多個**職責單一的 profile**，而非全部塞給單一 agent，能有效解決 context 污染、skill 膨脹、判斷力模糊三問題。Hermes 的實踐方式是 **Scout / Analyst / Briefer 三層管線**。

## 核心思想

- 單一 agent 同時承擔搜尋、分析、回報，三件事都會做不好
- 拆成三個 profile 後，各自保持小 context、單一職責
- 三個 profile 僅共享一個知識庫資料夾作為協調層

## 三個角色

| Profile | 工作 | 寫入 | 模型層級 |
|---------|------|------|---------|
| [scout-profile](./entities/scout-profile.md) | 定期蒐集 signal，不分析 | inbox 資料夾 | 便宜、快速 |
| [analyst-profile](./entities/analyst-profile.md) | 綜合 signal 成結構化知識 | 共享 wiki | 強推理 |
| [briefer-profile](./entities/briefer-profile.md) | 讀取 wiki，每天清晨發 briefing | Telegram | 便宜、快速 |

## 關鍵設計

### SOUL.md 極簡原則
每個 profile 的 SOUL.md 控制在 ~50 行內，**明確禁止**其他兩個角色的工作。

### wakeAgent gate
在排程 agent 真正啟動前，用一隻小腳本檢查是否有待處理檔案。inbox 為空則不喚醒 agent，避免付錢讓它看空氣。

### Obsidian + LLM Wiki 作為共享記憶
三個 profile 各自只能寫入自己的資料夾：
- Scout → inbox
- Analyst → sources / synthesis
- Briefer → 唯讀 synthesis，輸出到 Telegram

## 成本
大約 **US$20 ~ US$27 / 月**，前提是兩個輕型 profile 用便宜模型、Analyst 才用推理模型，且啟動 **wakeAgent gate**。用單一 API gateway 管理三個模型的 key，可進一步簡化。

## 適用場景
- 創辦人追蹤競爭對手
- 內容創作者每日研究單一領域
- 營運者監控多個市場
- 每天花 30 分鐘讀 Newsletter / 查五個分頁的人

## 常見錯誤
1. 在單一 profile 上堆疊新工作，而非新建 profile
2. 跳過 wakeAgent gate，讓空跑持續計費
3. 寫太長的 SOUL.md，每回合重讀耗 token
4. 所有 profile 都能寫入整个 vault，失去可 debug 性
5. 剛開始就用 kanban dispatcher，三層管線不需要

## 相關頁面
- [multi-agent-vs-single-agent](./comparisons/multi-agent-vs-single-agent.md)
- [scout-profile](./entities/scout-profile.md)
- [analyst-profile](./entities/analyst-profile.md)
- [briefer-profile](./entities/briefer-profile.md)

