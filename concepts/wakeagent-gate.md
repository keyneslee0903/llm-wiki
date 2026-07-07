---
title: wakeAgent Gate
created: 2026-07-08
updated: 2026-07-08
type: concept
tags: [technique, cost, inference]
sources: [raw/articles/mixroute-hermes-multi-agent-setup-2026-06-20.md]
confidence: high
---

# wakeAgent Gate

wakeAgent gate 是排程驅動的 Hermes cron 中，在 agent 啟動前先執行的小腳本，用途是決定「這次值不值得喚醒 agent」。

## 運作原理
1. cron 觸發後先跑 gate 腳本
2. 腳本檢查指定資料夾是否為空
3. 不為空 → output `{"wakeAgent": true}`，agent 啟動
4. 為空 → output `{"wakeAgent": false}`，agent 不啟動，不花錢

## 典型實作
```python
#!/usr/bin/env python3
import os, json

inbox = os.path.expanduser("~/research/inbox")
files = [f for f in os.listdir(inbox) if f.endswith('.md')] if os.path.exists(inbox) else []

if files:
    print(json.dumps({"wakeAgent": True}))
    print(f"{len(files)} new files in inbox:")
    for f in files:
        print(f"  {f}")
else:
    print(json.dumps({"wakeAgent": False}))
```

## 應用場景
- 搭配 Analyst 的每日 cron，只在 inbox 有檔案時才跑綜合
- 競品頁变化偵測：計算頁面 hash，只有 hash 改變時才喚醒
- 任何監控型 cron，避免「定時去看但沒事發生」的計費

## 成本影響
這是整套 multi-agent 系統能壓在 **US$20 ~ $27 / 月** 的關鍵機制。沒有它，三個 cron 每日都會啟動 agent，即便沒有任何新工作。

## 相關頁面
- [[hermes-multi-agent-setup]]
- [[analyst-profile]]
